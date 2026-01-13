## mbusd: Modbus TCP to RTU Gateway Daemon

## not used by any product

Based on my analysis, **mbusd** is a **production-grade Modbus gateway daemon** that bridges Modbus TCP networks with Modbus RTU (RS-232/485) serial networks. It's a **forked/open-source project** integrated into Hangtu's ecosystem to provide industrial communication capabilities.

### **What mbusd Has Done:**

#### 1. **Industrial Communication Gateway**
- **Protocol translation**: Converts Modbus TCP requests to Modbus RTU commands
- **Bidirectional communication**: Allows TCP masters to access RTU slaves through serial interfaces
- **Network abstraction**: Presents multiple RTU slaves as a single TCP slave/server

#### 2. **Core Architecture:**

**Gateway Pattern Implementation**
```
TCP Clients (Modbus TCP Masters)
        ↓
   mbusd (Gateway)
        ↓
Serial Port (RS-232/485)
        ↓
RTU Slaves (Modbus RTU Devices)
```

**Multi-Master Support**
- **Concurrent TCP connections**: Multiple TCP masters can access the same RTU network
- **Connection management**: Handles simultaneous requests with connection pooling
- **Request queuing**: Manages serial communication contention

#### 3. **Key Features Implemented:**

**A. Modbus Protocol Support**
- **Standard function codes**: 01, 02, 03, 04, 05, 06, 07, 15, 16
- **CRC-16 validation**: Hardware-accelerated CRC calculation for RTU frames
- **Error handling**: Retry logic for mismatched response CRC
- **Broadcast support**: Optional broadcast reply functionality

**B. Serial Communication**
- **Flexible RTU modes**: Configurable speed, parity, stop bits, timeouts
- **RS-485 support**: Automatic and manual direction control (RTS bit)
- **File-based control**: Alternative RS-485 control via file writes
- **Robust timing**: Configurable wait times and response timeouts

**C. System Integration**
- **Systemd service**: Native systemd integration with template services
- **Configuration management**: File-based configuration with examples
- **Logging system**: Configurable verbosity levels (0-9)
- **Daemon operation**: Proper daemonization with PID management

**D. Performance and Reliability**
- **Small footprint**: Suitable for embedded devices and SBCs (Raspberry Pi)
- **Connection limits**: Configurable maximum simultaneous connections
- **Retry mechanisms**: Configurable retry counts for failed requests
- **Timeout handling**: Connection and response timeouts

#### 4. **Technical Implementation:**

**Modbus Frame Processing**
```c
// modbus.c - Core Modbus protocol handling
int modbus_process_request(struct conn *conn, uint8_t *req, int reqlen);
int modbus_send_response(struct conn *conn, uint8_t *resp, int resplen);
```

**Serial Communication Layer**
```c
// tty.c - Serial port management
int tty_open(const char *device, int speed, char mode, int rts);
int tty_read(int fd, uint8_t *buf, size_t len, int timeout);
int tty_write(int fd, uint8_t *buf, size_t len);
```

**TCP Socket Management**
```c
// sock.c - Network communication
int sock_listen(const char *addr, int port, int maxconn);
int sock_accept(int listenfd, struct sockaddr *addr, socklen_t *addrlen);
```

#### 5. **Deployment and Packaging:**

**System Integration**
- **Debian packages**: `.deb` packaging for easy deployment
- **Docker support**: Containerized deployment option
- **Service management**: systemd unit files with instance support
- **Configuration templates**: Example configs in `/usr/local/etc/mbusd/`

**Multi-Instance Support**
```bash
# Run multiple instances for different serial ports
systemctl start mbusd@COM1.service
systemctl start mbusd@COM2.service
systemctl start mbusd@ttyUSB0.service
```

### **Comparison with Other SDKs:**

| Aspect | mbusd | htsdk | hwapi | Other SDKs |
|--------|-------|-------|-------|------------|
| **Domain** | **Industrial comms** | Hardware I/O | System monitoring | Various |
| **Type** | **Gateway daemon** | SDK library | API library | Libraries/SDKs |
| **Protocol** | **Modbus TCP/RTU** | Custom APIs | Custom APIs | Various |
| **Deployment** | **System service** | Linked library | Linked library | Various |
| **Origin** | **Open-source fork** | In-house | In-house | In-house/open-source |

### **Relationship with Other SDKs:**

#### **Complementary Industrial Stack**
```
┌─────────────────────────────────┐
│   Industrial Applications       │ ← Use htsdk for hardware control
├─────────────────────────────────┤
│       mbusd (This)              │ ← Provides Modbus connectivity
├─────────────────────────────────┤
│   htsdk/hwapi/hwio              │ ← Hardware access and monitoring
├─────────────────────────────────┤
│   Hardware (Serial, GPIO, etc.) │ ← Physical hardware
└─────────────────────────────────┘
```

#### **Integration Points**
- **htsdk's CAN/GPS/DIO**: Could be exposed via Modbus through mbusd
- **hwapi system monitoring**: Could provide Modbus registers for monitoring
- **Industrial SCADA systems**: Use mbusd to integrate Hangtu hardware into SCADA

### **What Makes mbusd Special:**

#### **Production-Ready Industrial Gateway**
- **Battle-tested**: Used in industrial environments
- **Standards-compliant**: Implements Modbus standard correctly
- **Robust error handling**: Industrial-grade reliability features

#### **Flexible Deployment Options**
- **Embedded friendly**: Small footprint for resource-constrained devices
- **Containerized**: Docker support for modern deployments
- **System integration**: Native systemd service management

#### **Open-Source Foundation**
- **Community maintained**: Benefits from upstream improvements
- **Security audited**: Open codebase allows security review
- **License compatibility**: BSD license allows commercial use

### **Technical Architecture:**

#### **Core Components**
1. **Main event loop** (`main.c`): Manages TCP connections and serial I/O
2. **Modbus protocol** (`modbus.c`): Frame parsing and validation
3. **Serial communication** (`tty.c`): RS-232/485 device handling
4. **TCP networking** (`sock.c`): Socket management
5. **Configuration** (`cfg.c`): Config file parsing
6. **Logging** (`log.c`): System logging

#### **Data Flow**
```
TCP Request → Parse → Queue → Serial Write → Wait → Serial Read → Validate → TCP Response
```

#### **Concurrency Model**
- **Single-threaded event loop**: Simple, reliable architecture
- **Non-blocking I/O**: Efficient handling of multiple connections
- **Request queuing**: Serializes access to shared serial port

### **Use Cases and Applications:**

#### **1. Industrial Automation**
- **SCADA integration**: Connect Hangtu hardware to SCADA systems via Modbus
- **PLC communication**: Interface with industrial PLCs
- **HMI connectivity**: Provide data to human-machine interfaces

#### **2. Building Management**
- **BACnet/Modbus gateways**: Bridge to building automation systems
- **Energy monitoring**: Expose power monitoring data via Modbus
- **Environmental controls**: Temperature, humidity monitoring

#### **3. IoT and Edge Computing**
- **Edge gateway**: Aggregate sensor data for cloud transmission
- **Protocol translation**: Convert between Modbus and modern IoT protocols
- **Legacy integration**: Connect legacy industrial equipment to modern systems

#### **4. Testing and Development**
- **Modbus device simulation**: Test Modbus client applications
- **Protocol debugging**: Analyze Modbus communication
- **Integration testing**: Verify system integration with Modbus devices

### **Integration with Hangtu's Ecosystem:**

#### **Hardware Compatibility**
- **Serial ports**: Works with Hangtu's RS-232/485 interfaces
- **GPIO control**: Could integrate with htsdk's DIO for RS-485 direction control
- **System monitoring**: Could expose hwapi metrics via Modbus registers

#### **Potential Enhancements**
```c
// Example: Expose hwapi metrics via Modbus
ModbusRegister hwapi_registers[] = {
    {40001, ht_cpu_temperature},    // CPU temperature
    {40002, ht_power_state},        // Power status
    {40003, ht_cpu_usage},          // CPU usage
    // ... more metrics
};
```

### **Development and Maintenance:**

#### **Upstream Relationship**
- **Fork of 3cky/mbusd**: Based on open-source project
- **Custom modifications**: Hangtu-specific enhancements
- **Upstream sync**: Can incorporate upstream improvements

#### **Quality Assurance**
- **CI/CD pipeline**: GitLab CI configuration
- **Automated testing**: Python test framework
- **Packaging**: Debian package generation

#### **Documentation**
- **Man pages**: `mbusd.8` manual page
- **Configuration examples**: Comprehensive example configs
- **Systemd integration**: Service management documentation

### **Limitations and Considerations:**

#### **Protocol Limitations**
- **Modbus only**: Single protocol support
- **Function code subset**: Not all Modbus function codes implemented
- **Vendor extensions**: Limited support for vendor-specific extensions

#### **Performance Characteristics**
- **Serial bottleneck**: Limited by serial port speed
- **Single serial port**: One RTU network per instance
- **Request serialization**: Concurrent requests queued for serial access

### **Conclusion:**

**mbusd** is a **critical industrial communication component** that:
- **Bridges modern TCP networks** with legacy serial industrial protocols
- **Enables SCADA/PLC integration** of Hangtu hardware
- **Provides standards-based connectivity** using widely adopted Modbus protocol
- **Complements Hangtu's hardware SDKs** with industrial communication capabilities

While **technically separate** from the core SDKs (htsdk, hwapi, hwio), **mbusd plays a vital role** in:
1. **Market expansion**: Enables entry into industrial automation markets
2. **System integration**: Allows integration with existing industrial systems
3. **Protocol standardization**: Provides industry-standard communication interface

**mbusd represents the "industrial connectivity layer"** of Hangtu's ecosystem, bridging the gap between Hangtu's proprietary hardware APIs and the standardized world of industrial automation. Its open-source foundation and production-ready implementation make it a valuable asset for deploying Hangtu hardware in industrial environments where Modbus is the lingua franca of device communication.

The integration of mbusd demonstrates Hangtu's commitment to **interoperability and standards compliance**, ensuring their hardware can participate in the broader industrial ecosystem while maintaining their proprietary advantages through htsdk and hwapi.
