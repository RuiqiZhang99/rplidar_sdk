Slamtec RPLIDAR Public SDK for C++
==================================

Introduction
------------

Slamtec激光雷达(https://www.slamtec.com/lidar/a3) 系列是一套高性能、低成本的激光雷达(https://en.wikipedia.org/wiki/Lidar)，是2D SLAM、3D重建、多点触摸和安全应用的完美传感器。
这是C++中RPLIDAR产品的公共SDK，并在GPLv3许可下开源。
如果您使用的是ROS（机器人操作系统），请直接使用我们的开源ROS节点：https://github.com/slamtec/rplidar_ros 。
如果您只是在评估RPLIDAR，您可以使用Slamtec RoboStudio(https://www.slamtec.com/robostudio （目前仅支持窗户）。
License
-------

SDK本身是根据BSD 2条款许可证进行许可的。
演示应用程序根据GPLv3许可证进行许可。

Release Notes
-------------

* [v1.12.0](https://github.com/slamtec/rplidar_sdk/blob/master/docs/ReleaseNote.v1.12.0.md)
* [v1.11.0](https://github.com/slamtec/rplidar_sdk/blob/master/docs/ReleaseNote.v1.11.0.md)
* [v1.10.0](https://github.com/slamtec/rplidar_sdk/blob/master/docs/ReleaseNote.v1.10.0.md)
* [v1.9.1](https://github.com/slamtec/rplidar_sdk/blob/master/docs/ReleaseNote.v1.9.1.md)
* [v1.9.0](https://github.com/slamtec/rplidar_sdk/blob/master/docs/ReleaseNote.v1.9.0.md)
* [v1.8.1](https://github.com/slamtec/rplidar_sdk/blob/master/docs/ReleaseNote.v1.8.1.md)
* [v1.8.0](https://github.com/slamtec/rplidar_sdk/blob/master/docs/ReleaseNote.v1.8.0.md)

Supported Platforms
-------------------

RPLIDAR SDK supports Windows, macOS and Linux by using Visual Studio 2010 projects and Makefile.

| LIDAR Model \ Platform | Windows | macOS | Linux |
| ---------------------- | ------- | ----- | ------|
| A1                     | Yes     | Yes   | Yes   |
| A2                     | Yes     | Yes   | Yes   |
| A3                     | Yes     | Yes   | Yes   |

Quick Start
-----------

### On Windows

如果安装了Microsoft Visual Studio 2010，只需打开sdk/workspace/vc10/sdk_和_demo.sln并编译即可。它包含库以及一些演示应用程序

### On macOS and Linux

请确保已安装make和g++，然后只需在根目录中调用make，即可在`output/$PLATFORM/$SCHEME`处获得编译结果，如`output/Linux/Release`。

    make

默认情况下，Makefile编译发布版本，您还可以使用`makedebug=1`编译调试版本。

Cross Compile
-------------

The Makefile system used by RPLIDAR public SDK support cross compiling.

The following command can be used to cross compile the SDK for `arm-linux-gnueabihf` targets:

    CROSS_COMPILE_PREFIX=arm-linux-gnueabihf ./cross_compile.sh

Demo Applications
-----------------

RPLIDAR public SDK includes some simple demos to do fast evaulation:

### ultra_simple

This demo application simply connects to an RPLIDAR device and outputs the scan data to the console.

    ultra_simple <serial_port_device>

For instance:

    ultra_simple \\.\COM11  # on Windows
    ultra_simple /dev/ttyUSB0

> Note: Usually you need root privilege to access tty devices under Linux. To eliminate this limitation, please add `KERNEL=="ttyUSB*", MODE="0666"` to the configuration of udev, and reboot.

### simple_grabber

This application demonstrates the process of getting RPLIDAR’s serial number, firmware version and healthy status after connecting the PC and RPLIDAR. Then the demo application grabs two round of scan data and shows the range data as histogram in the command line mode.

### frame_grabber (Legacy)

This demo application can show real-time laser scans in the GUI and is only available on Windows platform.

We have stopped the development of this demo application, please use Slamtec RoboStudio (https://www.slamtec.com/robostudio) instead.

SDK Usage
---------

> For detailed documents of RPLIDAR SDK, please refer to our user manual: https://download.slamtec.com/api/download/rplidar-sdk-manual/1.0?lang=en

### Include Header

    #include <rplidar.h>

Usually you only need to include this file to get all functions of RPLIDAR SDK.

### SDK Initialization and Termination

有两个静态接口用于创建和处理RPLIDAR驱动程序实例。每个RPLIDAR驱动程序实例只能用于与一个RPLIDAR设备通信。您可以自由分配任意数量的RPLIDAR驱动程序实例，以同时与多个RPLIDAR设备通信。
    /// Create an RPLIDAR Driver Instance
    /// This interface should be invoked first before any other operations
    ///
    /// \param drivertype the connection type used by the driver. 
    static RPlidarDriver * RPlidarDriver::CreateDriver(_u32 drivertype = DRIVER_TYPE_SERIALPORT);

    /// Dispose the RPLIDAR Driver Instance specified by the drv parameter
    /// Applications should invoke this interface when the driver instance is no longer used in order to free memory
    static void RPlidarDriver::DisposeDriver(RPlidarDriver * drv);

For example:

    #include <rplidar.h>

    int main(int argc, char* argv)
    {
        RPlidarDriver* lidar = RPlidarDriver::CreateDriver();

        // TODO

        RPlidarDriver::DisposeDriver(lidar);
    }

### Connect to RPLIDAR

创建RPlidarDriver实例后，可以使用`connect()`方法连接到串行端口:

    u_result res = lidar->connect("/dev/ttyUSB0", 115200);

    if (IS_OK(res))
    {
        // TODO
        lidar->disconnect();
    }
    else
    {
        fprintf(stderr, "Failed to connect to LIDAR %08x\r\n", res);
    }

### Start spinning motor

The LIDAR is not spinning by default. Method `startMotor()` is used to start this motor.

>对于RPLIDAR A1系列，该方法将使DTR信号使电机旋转；对于A2和A3系列，该方法将使附件板向电机的PWM引脚输出PWM信号。

    lidar->startMotor();
    // TODO
    lidar->stopMotor();

### Start scan

Slamtec RPLIDAR支持不同的扫描模式以实现兼容性和性能。自RPLIDAR SDK 1.6.0以来，SDK中添加了一个新的API`getAllSupportedScanModes()`。

    std::vector<RplidarScanMode> scanModes;
    lidar->getAllSupportedScanModes(scanModes);

您可以像这样从列表中选择扫描模式v:

    lidar->startScanExpress(false, scanModes[0].id);

或者你可以像这样使用RPLIDAR的典型扫描模式:

    RplidarScanMode scanMode;
    lidar->startScan(false, true, 0, &scanMode);

### Grab scan data
当RPLIDAR扫描时，您可以使用`GrabScanda()`和`GrabScandaHQ()` API获取一帧扫描。`GrabScanda()`和`GrabScandaHQ()`之间的区别是后者的支撑距离大于16.383m，这是RPLIDAR A2M6-R4和RPLIDAR A3系列所需的。
> `grabScanDataHq()`API与旧激光雷达模型和旧固件向后兼容。因此，我们建议始终使用此API，并仅为了兼容性而使用'`grabScanData()`。

    rplidar_response_measurement_node_hq_t nodes[8192];
    size_t nodeCount = sizeof(nodes)/sizeof(rplidar_response_measurement_node_hq_t);
    res = lidar->grabScanDataHq(nodes, nodeCount);

    if (IS_FAIL(res))
    {
        // failed to get scan data
    }

### Defination of data structure `rplidar_response_measurement_node_hq_t`

The defination of `rplidar_response_measurement_node_hq_t` is:

    #if defined(_WIN32)
    #pragma pack(1)
    #endif

    typedef struct rplidar_response_measurement_node_hq_t {
        _u16   angle_z_q14; 
        _u32   dist_mm_q2; 
        _u8    quality;  
        _u8    flag;
    } __attribute__((packed)) rplidar_response_measurement_node_hq_t;

    #if defined(_WIN32)
    #pragma pack()
    #endif

The definiton of each fields are:

| Field       | Data Type | Comments                                             |
| ----------- | --------- | -----------------------------------------------------|
| angle_z_q14 | u16_z_q14 | It is a fix-point angle desciption in z presentation |
| dist_mm_q2  | u32_q2    | Distance in millimeter of fixed point values         |
| quality     | u8        | Measurement quality (0 ~ 255)                        |
| flag        | u8        | Flags, current only one bit used: `RPLIDAR_RESP_MEASUREMENT_SYNCBIT` |

For example:

    float angle_in_degrees = node.angle_z_q14 * 90.f / (1 << 14);
    float distance_in_meters = node.dist_mm_q2 / 1000.f / (1 << 2);

Contact Slamtec
---------------

If you have any extra questions, please feel free to contact us at our support email:

    support AT slamtec DOT com
