# IMU \& Kalman

# 一\.理解kalman滤波器

先讲一个故事, 你有一辆遥控小车，它在直线上跑。你有两个方法知道它的位置：

1. **用速度估算位置**（比如小车每秒跑2米，初始位置是0米）。

*问题*：如果车轮打滑或加速，估算会有误差。

2. **用传感器测量位置**（比如GPS）。

*问题*：传感器可能有延迟或不精准。

单独用这两个方法都不够准，**卡尔曼滤波的作用就是结合两者的优点，得到一个更靠谱的结果！**

如果你更相信速度估算, 就给他更大的权重吧\.

---

### 卡尔曼滤波的步骤

#### 1\. **预测（用模型猜）**

- 假设上一秒小车在5米处，速度是2米/秒，预测现在应该在7米（5\+2×1）。

- 但你知道这个预测可能不准（比如车轮打滑），于是给预测结果一个“信心值”（比如误差±0\.5米）。

#### 2\. **测量（用传感器看）**

- 传感器显示现在位置是7\.5米，但传感器也有误差，比如“信心值”是±1米。

#### 3\. **结合预测和测量**

- **谁更可信？** 预测的误差更小（±0\.5米 \< ±1米），所以更相信预测。

- **最终结果**：在7米和7\.5米之间选一个值，但更靠近预测值（比如7\.2米）。

- **调整信心值**：结合后的结果误差比原来更小（比如±0\.4米），因为综合了两种信息。

#### 4\. **重复这个过程**

- 下一秒钟，用新的位置（7\.2米）和速度继续预测，再结合新的传感器数据，不断优化结果。这个步骤的详细解释可以看kalman\.c中的**void** **kalman**\(Kalman\_filter \*ekf, **float** input\)的注释

---

### 核心思想

- **预测和测量都不完美**，但可以互相弥补。

- **动态调整信任度**：谁误差小就更相信谁。

- **越融合越精准**：每次结合预测和测量后，结果的不确定性都会降低。





# 二\. mpu6050的具体驱动

```C
/*
 * mpu6050.h
 *  Created on: Jun 19, 2025
 *      Author: 张逸飞
 */

**#ifndef** INC_MPU6050_H_
**#define** INC_MPU6050_H_

**#include** "stm32f4xx_hal.h"
**#include** "main.h"
**#include** "i2c.h"
**#include** "math.h"
// 寄存器地址
**#define** MPU6050_ADDR 0xD0    // 从机地址(左移一位)因为要留一位来表示读or写
**#define** SMPLRT_DIV_REG 0x19 // 采样率
**#define** CONFIG 0x1A        //配置寄存器
**#define** GYRO_CONFIG_REG 0x1B  // 陀螺仪
**#define** ACCEL_CONFIG_REG 0x1C // 加速度
**#define** ACCEL_XOUT_H_REG 0x3B     //加速度数据寄存器, 按xyz顺序排列, 每个占2字节, 可以依次读值
**#define** TEMP_OUT_H_REG 0x41        //温度数据寄存器
**#define** GYRO_XOUT_H_REG 0x43    //陀螺仪数据寄存器, 与加速度同
**#define** PWR_MGMT_1_REG 0x6B   //电源管理寄存器
**#define** PWR_MGMT_2_REG 0x6C  //另一个电源管理寄存器
**#define** WHO_AM_I_REG 0x75 //读取此寄存器，若返回值为 0x68 则表示通信正常

**void** **mpu6050_init**(**void**);   //初始化
**void** **mpu6050_update**(**void**); // 读roll, pitch, yaw

// mpu6050类(也许没有必要)
**typedef** **struct**
{
        **void** (*init)(**void**);
        **void** (*update)(**void**);
} Mpu6050;

// 创建一个全局的MPU6050实例
**extern** Mpu6050 mpu6050;

// 导出到外部文件, 可以直接调用
**extern** **float** Ax, Ay, Az, Gx, Gy, Gz, roll, pitch, yaw; // A代表加速度，G代表角速度

**#endif** /* INC_MPU6050_H_ */

```

```C++
/*
 * mpu6050.c
 *
 *  Created on: Jun 19, 2025
 *      Author: 张逸飞
 */

**#include** "mpu6050.h"
**#include** "math.h"
**#include** <stdint.h>
**#include** "kalman.h"

// 定义6个int16_t的数据，一会用于对于接收的数据拼接
// int16_t是有负数的，uint16_t是没负数的
int16_t Accel_X_RAW = 0;
int16_t Accel_Y_RAW = 0;
int16_t Accel_Z_RAW = 0;
int16_t Gyro_X_RAW = 0;
int16_t Gyro_Y_RAW = 0;
int16_t Gyro_Z_RAW = 0;

// 这6个是把拼接好的数据经过计算得到结果
**float** Ax, Ay, Az, Gx, Gy, Gz;

// 使用加速度计算《加速度欧拉角》，先定义几个变量。_a的意思就是用的加速度
**float** roll_a, pitch_a;

// 使用角速度计算《角速度欧拉角》，先定义几个变量。_g的意思就是用的角速度
**float** roll_g, pitch_g, yaw_g;

// 欧拉角
**float** roll, pitch, yaw;

// 为每个轴创建卡尔曼滤波器
Kalman_filter kf_ax = {0.02, 0.0, 0.0, 0.0, 0.01, 0.1, kalman};
Kalman_filter kf_ay = {0.02, 0.0, 0.0, 0.0, 0.01, 0.1, kalman};
Kalman_filter kf_az = {0.02, 0.0, 0.0, 0.0, 0.01, 0.1, kalman};
Kalman_filter kf_gx = {0.02, 0.0, 0.0, 0.0, 0.01, 0.1, kalman};
Kalman_filter kf_gy = {0.02, 0.0, 0.0, 0.0, 0.01, 0.1, kalman};
Kalman_filter kf_gz = {0.02, 0.0, 0.0, 0.0, 0.01, 0.1, kalman};
Kalman_filter kf_roll = {0.02, 0.0, 0.0, 0.0, 0.01, 0.05, kalman};
Kalman_filter kf_pitch = {0.02, 0.0, 0.0, 0.0, 0.01, 0.05, kalman};
Kalman_filter kf_yaw = {0.02, 0.0, 0.0, 0.0, 0.01, 0.05, kalman};

// 全局MPU6050实例
Mpu6050 mpu6050 = {
                .init = mpu6050_init,
                .update = mpu6050_update};

**void** **mpu6050_init**(**void**) // 初始化
{
        uint8_t check;
        uint8_t Data;

        HAL_I2C_Mem_Read(&hi2c1, MPU6050_ADDR, WHO_AM_I_REG, 1, &check, 1, 1000);
        **if** (check == 0x68) 
        {
                // 电源管理1
                Data = 0x01;
                HAL_I2C_Mem_Write(&hi2c1, MPU6050_ADDR, PWR_MGMT_1_REG, 1, &Data, 1, 1000);

                // 电源管理2
                Data = 0x00;
                HAL_I2C_Mem_Write(&hi2c1, MPU6050_ADDR, PWR_MGMT_2_REG, 1, &Data, 1, 1000);

                // 滤波
                Data = 0x06;
                HAL_I2C_Mem_Write(&hi2c1, MPU6050_ADDR, CONFIG, 1, &Data, 1, 1000);

                // 采样频率分频器寄存器
                Data = 0x09;
                HAL_I2C_Mem_Write(&hi2c1, MPU6050_ADDR, SMPLRT_DIV_REG, 1, &Data, 1, 1000);

                // 加速度计配置
                Data = 0x18;
                HAL_I2C_Mem_Write(&hi2c1, MPU6050_ADDR, ACCEL_CONFIG_REG, 1, &Data, 1, 1000);

                // 陀螺仪配置
                Data = 0x18;
                HAL_I2C_Mem_Write(&hi2c1, MPU6050_ADDR, GYRO_CONFIG_REG, 1, &Data, 1, 1000);
        }

        // 初始化欧拉角
        roll = 0;
        pitch = 0;
        yaw = 0;
        yaw_g = 0; // 初始化yaw角速度积分值
}

// 此函数是读取结果，当然这个结果是卡尔曼滤波法计算的
**void** **mpu6050_update**(**void**)
{
        // 用于陀螺仪零点校准
        **static** **float** gyro_z_offset = 0;
        **static** **int** calibration_count = 0;
        **static** **float** gyro_z_sum = 0;
        **const** **int** CALIBRATION_SAMPLES = 100;

        // 用于时间计算
        **static** uint32_t last_time = 0;
        **const** **float** dt = 0.005f; // 默认时间间隔，单位秒

        // 计算实际时间间隔
        uint32_t current_time = HAL_GetTick();
        **float** time_diff;
        **if** (last_time == 0)
        {
                time_diff = dt;
        }
        **else**
        {
                time_diff = (current_time - last_time) / 1000.0f; // 转换为秒
        }
        last_time = current_time;

        // 读取加速度计数据
        uint8_t Rec_Data_A[6];
        HAL_I2C_Mem_Read(&hi2c1, MPU6050_ADDR, ACCEL_XOUT_H_REG, 1, Rec_Data_A, 6, 1000);
        Accel_X_RAW = (int16_t)(Rec_Data_A[0] << 8 | Rec_Data_A[1]);
        Accel_Y_RAW = (int16_t)(Rec_Data_A[2] << 8 | Rec_Data_A[3]);
        Accel_Z_RAW = (int16_t)(Rec_Data_A[4] << 8 | Rec_Data_A[5]);

        // 转换原始数据
        **float** ax_raw = Accel_X_RAW / 2048.0;
        **float** ay_raw = Accel_Y_RAW / 2048.0;
        **float** az_raw = Accel_Z_RAW / 2048.0;

        // 对加速度数据应用卡尔曼滤波
        kf_ax.filt(&kf_ax, ax_raw);
        kf_ay.filt(&kf_ay, ay_raw);
        kf_az.filt(&kf_az, az_raw);

        // 使用滤波后的数据
        Ax = kf_ax.out;
        Ay = kf_ay.out;
        Az = kf_az.out;

        // 读取陀螺仪数据
        uint8_t Rec_Data_G[6];
        HAL_I2C_Mem_Read(&hi2c1, MPU6050_ADDR, GYRO_XOUT_H_REG, 1, Rec_Data_G, 6, 1000);
        Gyro_X_RAW = (int16_t)(Rec_Data_G[0] << 8 | Rec_Data_G[1]);
        Gyro_Y_RAW = (int16_t)(Rec_Data_G[2] << 8 | Rec_Data_G[3]);
        Gyro_Z_RAW = (int16_t)(Rec_Data_G[4] << 8 | Rec_Data_G[5]);

        // 转换原始数据
        **float** gx_raw = Gyro_X_RAW / 16.384;
        **float** gy_raw = Gyro_Y_RAW / 16.384;
        **float** gz_raw = Gyro_Z_RAW / 16.384;

        // 对陀螺仪数据应用卡尔曼滤波
        kf_gx.filt(&kf_gx, gx_raw);
        kf_gy.filt(&kf_gy, gy_raw);
        kf_gz.filt(&kf_gz, gz_raw);

        // 使用滤波后的数据
        Gx = kf_gx.out;
        Gy = kf_gy.out;
        Gz = kf_gz.out;

        // 陀螺仪零点校准
        **if** (calibration_count < CALIBRATION_SAMPLES)
        {
                gyro_z_sum += Gz;
                calibration_count++;

                **if** (calibration_count == CALIBRATION_SAMPLES)
                {
                        gyro_z_offset = gyro_z_sum / CALIBRATION_SAMPLES;
                }

                // 校准期间不更新欧拉角
                **if** (calibration_count <= CALIBRATION_SAMPLES)
                {
                        **return**;
                }
        }

        // 应用零点校正
        Gz -= gyro_z_offset;

        // 从加速度计算roll和pitch欧拉角
        **float** roll_acc = **atan2**(Ay, Az) * 57.295779513f; // 180/PI = 57.295779513
        **float** pitch_acc = -**atan2**(Ax, **sqrt**(Ay * Ay + Az * Az)) * 57.295779513f;

        // 对欧拉角应用卡尔曼滤波
        kf_roll.filt(&kf_roll, roll_acc);
        kf_pitch.filt(&kf_pitch, pitch_acc);

        // 使用滤波后的欧拉角
        roll = kf_roll.out;
        pitch = kf_pitch.out;

        // 使用陀螺仪积分计算yaw角度
        yaw_g += Gz * time_diff;

        // 将yaw角度限制在±180度范围内
        **if** (yaw_g > 180.0f)
                yaw_g -= 360.0f;
        **if** (yaw_g < -180.0f)
                yaw_g += 360.0f;

        // 对yaw角度应用卡尔曼滤波
        kf_yaw.filt(&kf_yaw, yaw_g);

        // 使用滤波后的yaw角度
        yaw = kf_yaw.out;

        // 限制yaw在±90度范围内用于显示
        **if** (yaw > 90.0f)
                yaw = 90.0f;
        **if** (yaw < -90.0f)
                yaw = -90.0f;
}

```

调用示例

```C++
//删除了很多必要的但是和mpu6050无关的代码
**#include** "mpu6050.h"
**#include** "kalman.h"

**int** **main**(**void**)
{
  // 初始化I2C1
  MX_I2C1_Init();
  //创建调试信息缓冲区
  char buffer[100];
  // 初始化MPU6050, 这个在mpu6050.h已经声明过extern了, 可以直接用
  mpu6050.init();
  **while** (1)
  {
    // 读取MPU6050的数据
    mpu6050.update();
    **sprintf**(buffer, "Roll: %.2f, Pitch: %.2f, Yaw: %.2f\r\n", roll, pitch, yaw);
    HAL_UART_Transmit(&huart1, (uint8_t *)buffer, **strlen**(buffer), HAL_MAX_DELAY);
    HAL_Delay(100);
  }
```

# 三\. Kalman滤波器

```C++
/*
 * kalman.h
 *
 *  Created on: Jun 21, 2025
 *      Author: Allen
 */

**#ifndef** INC_KALMAN_H_
**#define** INC_KALMAN_H_

// 前向声明结构体类型
**typedef** **struct** _Kalman_filter Kalman_filter;

// 卡尔曼滤波结构体
**struct** _Kalman_filter
{
        **float** LastP; // 前序协方差
        **float** NowP;         // 当前协方差
        **float** out;         // 滤波结果
        **float** Kg;                 // 卡尔曼增益
        **float** Q;                 // 背景白噪音
        **float** R;                 // 器件方差
        **void** (*filt)(Kalman_filter *ekf, **float** input);
};

**extern** Kalman_filter imu_filter;

// 一维卡尔曼
**void** **kalman**(Kalman_filter *ekf, **float** input);

**#endif** /* INC_KALMAN_H_ */

```

```SQL
/*
 * kalman.c
 *
 *  Created on: Jun 21, 2025
 *      Author: 张逸飞
 *  这是卡尔曼滤波的实现
 */

**#include** "kalman.h"

Kalman_filter imu_filter = {
                .LastP = 0.02,
                .NowP = 0.0,
                .out = 0.0,
                .Kg = 0.0,
                .Q = 0.08,
                .R = 0.1,
                .filt = kalman};

**void** **kalman**(Kalman_filter *ekf, **float** input)
{
        //预测位置 = 上一时刻滤波位置 + 速度 × 时间间隔
        ekf->NowP = ekf->LastP + ekf->Q;
        
        //如果 NowP较小，说明预测比较可靠，Kg 会较小，更倾向于相信预测值；
        //如果 R 较小，说明测量比较可靠，Kg会较大，更倾向于相信测量值。
        ekf->Kg = ekf->NowP / (ekf->NowP + ekf->R);
        
        //这里 ekf->out 是上一时刻滤波后的位置（7.2 米），input 是传感器测量的当前位置（9.5 米）。
        //经过计算得到的新的 ekf->out 就是当前时刻滤波后的更准确的位置。
        ekf->out = ekf->out + ekf->Kg * (input - ekf->out);
        
        //更新数据
        ekf->LastP = (1 - ekf->Kg) * ekf->NowP;
}

```



调用示例

```Plain Text
// 为每个轴创建卡尔曼滤波器
Kalman_filter kf_ax = {0.02, 0.0, 0.0, 0.0, 0.01, 0.1, kalman};
Kalman_filter kf_ay = {0.02, 0.0, 0.0, 0.0, 0.01, 0.1, kalman};
Kalman_filter kf_az = {0.02, 0.0, 0.0, 0.0, 0.01, 0.1, kalman};

// 读取加速度计数据
uint8_t Rec_Data_A[6];
HAL_I2C_Mem_Read(&hi2c1, MPU6050_ADDR, ACCEL_XOUT_H_REG, 1, Rec_Data_A, 6, 1000);
Accel_X_RAW = (int16_t)(Rec_Data_A[0] << 8 | Rec_Data_A[1]);
Accel_Y_RAW = (int16_t)(Rec_Data_A[2] << 8 | Rec_Data_A[3]);
Accel_Z_RAW = (int16_t)(Rec_Data_A[4] << 8 | Rec_Data_A[5]);

// 转换原始数据
**float** ax_raw = Accel_X_RAW / 2048.0;
**float** ay_raw = Accel_Y_RAW / 2048.0;
**float** az_raw = Accel_Z_RAW / 2048.0;

// 对加速度数据应用卡尔曼滤波
kf_ax.filt(&kf_ax, ax_raw);
kf_ay.filt(&kf_ay, ay_raw);
kf_az.filt(&kf_az, az_raw);

// 得到滤波后的数据
Ax = kf_ax.out;
Ay = kf_ay.out;
Az = kf_az.out;
```

[只有互补滤波\.mp4](../media/06%20SLAM%20定位导航/IMU%20&%20Kalman/只有互补滤波.mp4)

上面是只有互补滤波的情况, 响应较快但是噪声也相应的较多

[互补滤波\+kalman\.mp4](../media/06%20SLAM%20定位导航/IMU%20&%20Kalman/互补滤波+kalman.mp4)

这个是加上kalman的结果, 明显平滑了, 但是计算量大\( 甚至用的还不是kalman的完全体\) 导致响应没那么及时, 不过还可以接受



以上\.

---

2025\.6\.22

# 四\. 一些问题

## 为什么六轴无法解算Yaw?

加速度计可以等效为这个:


![image\.png](../media/06%20SLAM%20定位导航/IMU%20&%20Kalman/image.png)

当静止不动时, 加速度计读出的向量是$\begin{bmatrix}
0\\
0\\
-g
\end{bmatrix}$

当他的Roll角发生变化时, 读出的向量是$\begin{bmatrix}
0\\
a_y\\
a_x
\end{bmatrix}$

解算出角度:

$θ = \arctan(\frac{a_y}{a_z})$

![image\.png](../media/06%20SLAM%20定位导航/IMU%20&%20Kalman/image%201.png)

pitch角同理, 但是, 当yaw发生变化时, 获得的加速度始终是$\begin{bmatrix}
0 \\
0 \\
-g
\end{bmatrix}$, 加速度计不能获得Yaw的信息, Yaw的值完全来自于角速度计

## 角速度计\( 陀螺仪\)如何工作

角速度计可以获得绕x, y, z轴的角速度, 即Yaw, Pitch, Roll的角速度\.

$θ(t)=θ(0)+∫_0^tω(t)dt$, 即初始角度加上角速度的定积分

## Yaw为什么会零偏

- 偏置（Bias）陀螺仪零点漂移，即没动时也有小角速度输出，比如 0\.5°/s

- 噪声（Noise）测量存在高频抖动，会累积成随机游走（Random Walk）误差

假设有初始漂移0\.5 °/s, 不存在噪声, 那么这个漂移积分出来的误差再1分钟后就是30°, 这很恐怖

## Pitch和Roll为什么不会零偏

通常, 我们将陀螺仪数据和加速度计的数据进行互补滤波

$θ(t)=α⋅(θgyro)+(1−α)⋅(θacc)
$, α的大小决定了我们更相信角速度计还是加速度计

在这之中, 角速度计提供了快速响应的数据, 加速度计保证了不会偏移严重

## 磁力计纠正Yaw的原理是什么?

磁力计可以获得在本地坐标系下的磁感应强度

$\mathbf{B}_{local} =\begin{bmatrix} B_x\\B_y\\ B_z\end{bmatrix}$, 方向指向地磁北极

想象你拿着一个指南针, 初始方位角$Yaw_0=Roll_0=Pitch_0 =0$, 不妨认为他是水平的, 然后你走上了一个不规则的斜坡, 有了方位角$Yaw_1, Roll_1, Pitch_1$

由于上述的问题, 我们不能得到$Yaw_1$但能得到相当准确的$Roll_1和Pitch_1$

现在指南针的方向大致是向南的, 但是你的指针已经不在地图上了, 如果指南针要发挥作用, 它应当于地图面平行, 所以应当转$-Roll_1和-Pitch_1$, 这样指南针就回到平面了, 而$Yaw_1$无论是多少都不会让指南针脱离水平面, 此时, 指南针的指向就是标准地磁南

具体可以看这个视频:

[e0fc4ee931fcdc4a25eb7da98eeba009\.mp4](../media/06%20SLAM%20定位导航/IMU%20&%20Kalman/e0fc4ee931fcdc4a25eb7da98eeba009.mp4)



## 如何实现磁力计纠正Yaw?

![fb3f765f53b1a9e15d0b2a7646e3b0b\.jpg](../media/06%20SLAM%20定位导航/IMU%20&%20Kalman/fb3f765f53b1a9e15d0b2a7646e3b0b.jpg)

由mpu6050的结算过程, 我们知道$θ_{pitch}=\arctan(\frac{g_z}{\sqrt{g_x^2+g_y^2}} ), \theta_{roll}=\arctan(\frac{g_y}{g_z})$

对于三维空间的任意角度的旋转, 我们有旋转矩阵:

绕x轴旋转 $Roll(\theta)=\begin{bmatrix}
 1 & 0 & 0 \\
 0 & \cos(\theta) & -\sin(\theta) \\
 0 & \sin(\theta) & \cos(\theta) 
\end{bmatrix}$

绕y轴旋转 $Pitch(\theta)=\begin{bmatrix}
 \cos(\theta) & 0 & \sin(\theta) \\
 0 & 1 & 0 \\
 -\sin(\theta) & 0 & \cos(\theta) 
\end{bmatrix}$

绕z轴旋转 $Yaw(\theta)=\begin{bmatrix}
 \cos(\theta) & -\sin(\theta) & 0 \\
 \sin(\theta) & \cos(\theta) & 0 \\
 0 & 0 & 1 \\
\end{bmatrix}$

并且他们有以下良好的性质

- 三维旋转矩阵都是正交阵 $A^T = A^{-1}$

- 三维旋转矩阵的行列式都为1 $\begin{vmatrix}
 A
\end{vmatrix} = 1
$



对于三维向量$\alpha=\begin{bmatrix}
 a_1\\
 a_2\\
 a_3
\end{bmatrix}$, 绕x轴旋转$\theta_1$度后的向量$\alpha'=Roll(\theta_1)\alpha$, 即左乘相应角度的旋转矩阵即可

**应当指出, 由于矩阵乘法不满足交换律, 所以旋转顺序是不可忽视的**

但是这没有关系, 我们定义的姿态角有旋转顺序的要求, 是

$Yaw(\theta) $\-\>$Pitch(\theta)$\-\>$Roll(\theta
)$, 也称 **body\-321** 顺序

所以我们只需要对向量B 先左乘$Roll(-\theta)$再左乘$Pitch(-\theta)$, 得到$B'=\begin{bmatrix}
 B_x' \\
 B_y' \\
 B_z'
\end{bmatrix}$, 此时

$Yaw=\arctan(\frac{B_y'}{B_x'})$

还有一点, 就是这个Yaw是偏离地磁北的大小, 是绝对Yaw而非我们的相对Yaw\. 我们在进行姿态结算时习惯将初始化状态下的Yaw, Pitch和Roll进行归零操作, 这需要我们吧初始化的Yaw用一个变量保存起来, 这时:

$Yaw_{final}=\arctan(\frac{B_y'}{B_z'}) - Yaw_{start}$

## 九轴IMU中Yaw的解算

看起来我们刚刚用磁力计获得了绝对正确的值, 但是由于传感器噪声的存在, 这个值总归不是完全准确的\.

还记得我们的加速度计吗? $\theta(t)=θ(0)+∫_0^tω(t)dt$

一个不准确的传感器, 一个不准确的预测模型

这就是**Kalman Filter** 大显身手的地方

以上

---

2025\.7\.15

# 五\. 将初始化数据写入FLASH

现在每次上电都需要等待几秒钟来进行mpu6050的校准, 这很消耗我们的耐心, 而且这个校准的量通常变化不大, 对于这样的数据我们的解决方案是将他写道FLASH里

## FLASH是什么

是stm32放置程序和常量的地方, 掉电仍保存, 读写较慢

和FLASH地位相当的是SRAM, 用于放置变量和堆栈, 但是掉电就消失, 读写很快

## 读写FLASH\(以STM32F407VET6为例\)

STM32F407VET6的闪存是按扇区\(sector\)或页来区分的

```C
#ifndef __FLASH_UTIL_H
#define __FLASH_UTIL_H

#include "stm32f4xx_hal.h"
//向FLASH写入浮点数
void Flash_Write_Float(uint32_t address, float data);
//在FLASH读浮点数
float Flash_Read_Float(uint32_t address);

#endif
```

```C++
#include "flash_util.h"

void Flash_Write_Float(uint32_t address, float data)
{
    //解除FLASH写保护
    HAL_FLASH_Unlock();

    //声明擦除配置结构体 eraseInitStruct，以及错误记录变量 SectorError。
    FLASH_EraseInitTypeDef eraseInitStruct;
    uint32_t SectorError;
    
    //指定擦除类型为**按扇区擦除**
    eraseInitStruct.TypeErase = FLASH_TYPEERASE_SECTORS;
    
    //设置擦除时的电压范围，**3 代表2.7~3.6V**
    eraseInitStruct.VoltageRange = FLASH_VOLTAGE_RANGE_3;
    
    //指定要擦除的 Flash 扇区，这里是扇区 11（起始地址是 0x080E0000）
    eraseInitStruct.Sector = FLASH_SECTOR_11;
    
    //设置需要擦除的扇区数为 1
    eraseInitStruct.NbSectors = 1;

    //删除指定扇区, 返回值是是否报错
    HAL_FLASHEx_Erase(&eraseInitStruct, &SectorError);

    // 写入 float 类型数据, float恰好是4字节32bit
    HAL_FLASH_Program(FLASH_TYPEPROGRAM_WORD, address, *(uint32_t*)&data);
    
    //写入魔术字(表示写入成功)
    HAL_FLASH_Program(FLASH_TYPEPROGRAM_WORD, FLASH_YAW_FLAG_ADDR, FLASH_YAW_FLAG_VALUE);

    //FLASH写锁定
    HAL_FLASH_Lock();
}

float Flash_Load_YawOffset(float default_value)
{
    uint32_t flag = *(uint32_t*)FLASH_YAW_FLAG_ADDR;
    if (flag == FLASH_YAW_FLAG_VALUE)
    {
        uint32_t raw = *(uint32_t*)FLASH_YAW_OFFSET_ADDR;
        return *(float*)&raw;
    }
    else
    {
        return default_value;  // 未写入时返回默认值
    }
}


```

## 在mpu6050中使用FLASH写入

```C
/*
 * mpu6050.h
 *  Created on: Jun 19, 2025
 *      Author: 张逸飞
 */

**#ifndef** INC_MPU6050_H_
**#define** INC_MPU6050_H_

**#include** "stm32f4xx_hal.h"
**#include** "main.h"
**#include** "i2c.h"
**#include** "math.h"

#include"flash_util.h"

//要写入的FLASH扇区首地址, 这里是sector 11
#define FLASH_YAW_OFFSET_ADDR  0x080E0000  // 存 float yaw_offset
#define FLASH_YAW_FLAG_ADDR    0x080E0004  // 存是否写入的标志（magic number）
#define FLASH_YAW_FLAG_VALUE   0xDEADBEEF  // 你自定义的魔术字


// 寄存器地址
**#define** MPU6050_ADDR 0xD0    // 从机地址(左移一位)因为要留一位来表示读or写
**#define** SMPLRT_DIV_REG 0x19 // 采样率
**#define** CONFIG 0x1A        //配置寄存器
**#define** GYRO_CONFIG_REG 0x1B  // 陀螺仪
**#define** ACCEL_CONFIG_REG 0x1C // 加速度
**#define** ACCEL_XOUT_H_REG 0x3B     //加速度数据寄存器, 按xyz顺序排列, 每个占2字节, 可以依次读值
**#define** TEMP_OUT_H_REG 0x41        //温度数据寄存器
**#define** GYRO_XOUT_H_REG 0x43    //陀螺仪数据寄存器, 与加速度同
**#define** PWR_MGMT_1_REG 0x6B   //电源管理寄存器
**#define** PWR_MGMT_2_REG 0x6C  //另一个电源管理寄存器
**#define** WHO_AM_I_REG 0x75 //读取此寄存器，若返回值为 0x68 则表示通信正常

**void** **mpu6050_init**(**void**);   //初始化
**void** **mpu6050_update**(**void**); // 读roll, pitch, yaw

// mpu6050类(也许没有必要)
**typedef** **struct**
{
        **void** (*init)(**void**);
        **void** (*update)(**void**);
} Mpu6050;

// 创建一个全局的MPU6050实例
**extern** Mpu6050 mpu6050;

// 导出到外部文件, 可以直接调用
**extern** **float** Ax, Ay, Az, Gx, Gy, Gz, roll, pitch, yaw; // A代表加速度，G代表角速度

**#endif** /* INC_MPU6050_H_ */

```

```C++
/*
 * mpu6050.c
 *
 *  Created on: Jun 19, 2025
 *      Author: 张逸飞
 */

**#include** "mpu6050.h"
**#include** "math.h"
**#include** <stdint.h>
**#include** "kalman.h"

// 定义6个int16_t的数据，一会用于对于接收的数据拼接
// int16_t是有负数的，uint16_t是没负数的
int16_t Accel_X_RAW = 0;
int16_t Accel_Y_RAW = 0;
int16_t Accel_Z_RAW = 0;
int16_t Gyro_X_RAW = 0;
int16_t Gyro_Y_RAW = 0;
int16_t Gyro_Z_RAW = 0;

// 这6个是把拼接好的数据经过计算得到结果
**float** Ax, Ay, Az, Gx, Gy, Gz;

// 使用加速度计算《加速度欧拉角》，先定义几个变量。_a的意思就是用的加速度
**float** roll_a, pitch_a;

// 使用角速度计算《角速度欧拉角》，先定义几个变量。_g的意思就是用的角速度
**float** roll_g, pitch_g, yaw_g;

// 欧拉角
**float** roll, pitch, yaw;

float gyro_z_offset; 

// 为每个轴创建卡尔曼滤波器
Kalman_filter kf_ax = {0.02, 0.0, 0.0, 0.0, 0.01, 0.1, kalman};
Kalman_filter kf_ay = {0.02, 0.0, 0.0, 0.0, 0.01, 0.1, kalman};
Kalman_filter kf_az = {0.02, 0.0, 0.0, 0.0, 0.01, 0.1, kalman};
Kalman_filter kf_gx = {0.02, 0.0, 0.0, 0.0, 0.01, 0.1, kalman};
Kalman_filter kf_gy = {0.02, 0.0, 0.0, 0.0, 0.01, 0.1, kalman};
Kalman_filter kf_gz = {0.02, 0.0, 0.0, 0.0, 0.01, 0.1, kalman};
Kalman_filter kf_roll = {0.02, 0.0, 0.0, 0.0, 0.01, 0.05, kalman};
Kalman_filter kf_pitch = {0.02, 0.0, 0.0, 0.0, 0.01, 0.05, kalman};
Kalman_filter kf_yaw = {0.02, 0.0, 0.0, 0.0, 0.01, 0.05, kalman};

// 全局MPU6050实例
Mpu6050 mpu6050 = {
                .init = mpu6050_init,
                .update = mpu6050_update};

**void** **mpu6050_init**(**void**) // 初始化
{
        uint8_t check;
        uint8_t Data;

        HAL_I2C_Mem_Read(&hi2c1, MPU6050_ADDR, WHO_AM_I_REG, 1, &check, 1, 1000);
        **if** (check == 0x68) 
        {
                // 电源管理1
                Data = 0x01;
                HAL_I2C_Mem_Write(&hi2c1, MPU6050_ADDR, PWR_MGMT_1_REG, 1, &Data, 1, 1000);

                // 电源管理2
                Data = 0x00;
                HAL_I2C_Mem_Write(&hi2c1, MPU6050_ADDR, PWR_MGMT_2_REG, 1, &Data, 1, 1000);

                // 滤波
                Data = 0x06;
                HAL_I2C_Mem_Write(&hi2c1, MPU6050_ADDR, CONFIG, 1, &Data, 1, 1000);

                // 采样频率分频器寄存器
                Data = 0x09;
                HAL_I2C_Mem_Write(&hi2c1, MPU6050_ADDR, SMPLRT_DIV_REG, 1, &Data, 1, 1000);

                // 加速度计配置
                Data = 0x18;
                HAL_I2C_Mem_Write(&hi2c1, MPU6050_ADDR, ACCEL_CONFIG_REG, 1, &Data, 1, 1000);

                // 陀螺仪配置
                Data = 0x18;
                HAL_I2C_Mem_Write(&hi2c1, MPU6050_ADDR, GYRO_CONFIG_REG, 1, &Data, 1, 1000);
        }

        // 初始化欧拉角
        roll = 0;
        pitch = 0;
        yaw = 0;
        yaw_g = 0; // 初始化yaw角速度积分值
        gyro_z_offset = Flash_Load_YawOffset(0.0f);  // 如果没写入过就用0

}

// 此函数是读取结果，当然这个结果是卡尔曼滤波法计算的
**void** **mpu6050_update**(**void**)
{
        // 用于陀螺仪零点校准
        **static** **int** calibration_count = 0;
        **static** **float** gyro_z_sum = 0;
        **const** **int** CALIBRATION_SAMPLES = 100;

        // 用于时间计算
        **static** uint32_t last_time = 0;
        **const** **float** dt = 0.005f; // 默认时间间隔，单位秒

        // 计算实际时间间隔
        uint32_t current_time = HAL_GetTick();
        **float** time_diff;
        **if** (last_time == 0)
        {
                time_diff = dt;
        }
        **else**
        {
                time_diff = (current_time - last_time) / 1000.0f; // 转换为秒
        }
        last_time = current_time;

        // 读取加速度计数据
        uint8_t Rec_Data_A[6];
        HAL_I2C_Mem_Read(&hi2c1, MPU6050_ADDR, ACCEL_XOUT_H_REG, 1, Rec_Data_A, 6, 1000);
        Accel_X_RAW = (int16_t)(Rec_Data_A[0] << 8 | Rec_Data_A[1]);
        Accel_Y_RAW = (int16_t)(Rec_Data_A[2] << 8 | Rec_Data_A[3]);
        Accel_Z_RAW = (int16_t)(Rec_Data_A[4] << 8 | Rec_Data_A[5]);

        // 转换原始数据
        **float** ax_raw = Accel_X_RAW / 2048.0;
        **float** ay_raw = Accel_Y_RAW / 2048.0;
        **float** az_raw = Accel_Z_RAW / 2048.0;

        // 对加速度数据应用卡尔曼滤波
        kf_ax.filt(&kf_ax, ax_raw);
        kf_ay.filt(&kf_ay, ay_raw);
        kf_az.filt(&kf_az, az_raw);

        // 使用滤波后的数据
        Ax = kf_ax.out;
        Ay = kf_ay.out;
        Az = kf_az.out;

        // 读取陀螺仪数据
        uint8_t Rec_Data_G[6];
        HAL_I2C_Mem_Read(&hi2c1, MPU6050_ADDR, GYRO_XOUT_H_REG, 1, Rec_Data_G, 6, 1000);
        Gyro_X_RAW = (int16_t)(Rec_Data_G[0] << 8 | Rec_Data_G[1]);
        Gyro_Y_RAW = (int16_t)(Rec_Data_G[2] << 8 | Rec_Data_G[3]);
        Gyro_Z_RAW = (int16_t)(Rec_Data_G[4] << 8 | Rec_Data_G[5]);

        // 转换原始数据
        **float** gx_raw = Gyro_X_RAW / 16.384;
        **float** gy_raw = Gyro_Y_RAW / 16.384;
        **float** gz_raw = Gyro_Z_RAW / 16.384;

        // 对陀螺仪数据应用卡尔曼滤波
        kf_gx.filt(&kf_gx, gx_raw);
        kf_gy.filt(&kf_gy, gy_raw);
        kf_gz.filt(&kf_gz, gz_raw);

        // 使用滤波后的数据
        Gx = kf_gx.out;
        Gy = kf_gy.out;
        Gz = kf_gz.out;

        // 陀螺仪零点校准
        **if** (calibration_count < CALIBRATION_SAMPLES)
        {
                gyro_z_sum += Gz;
                calibration_count++;

               if (calibration_count == CALIBRATION_SAMPLES)
                    {
                        gyro_z_offset = gyro_z_sum / CALIBRATION_SAMPLES;
                        Flash_Save_YawOffset(gyro_z_offset);  // 保存校准值
                    }

                // 校准期间不更新欧拉角
                **if** (calibration_count <= CALIBRATION_SAMPLES)
                {
                        **return**;
                }
        }

        // 应用零点校正
        Gz -= gyro_z_offset;

        // 从加速度计算roll和pitch欧拉角
        **float** roll_acc = **atan2**(Ay, Az) * 57.295779513f; // 180/PI = 57.295779513
        **float** pitch_acc = -**atan2**(Ax, **sqrt**(Ay * Ay + Az * Az)) * 57.295779513f;

        // 对欧拉角应用卡尔曼滤波
        kf_roll.filt(&kf_roll, roll_acc);
        kf_pitch.filt(&kf_pitch, pitch_acc);

        // 使用滤波后的欧拉角
        roll = kf_roll.out;
        pitch = kf_pitch.out;

        // 使用陀螺仪积分计算yaw角度
        yaw_g += Gz * time_diff;

        // 将yaw角度限制在±180度范围内
        **if** (yaw_g > 180.0f)
                yaw_g -= 360.0f;
        **if** (yaw_g < -180.0f)
                yaw_g += 360.0f;

        // 对yaw角度应用卡尔曼滤波
        kf_yaw.filt(&kf_yaw, yaw_g);

        // 使用滤波后的yaw角度
        yaw = kf_yaw.out;

        // 限制yaw在±90度范围内用于显示
        **if** (yaw > 90.0f)
                yaw = 90.0f;
        **if** (yaw < -90.0f)
                yaw = -90.0f;
}

```

---

以上

2025\.7\.25

