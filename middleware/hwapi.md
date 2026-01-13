## hwapi: High-Level Hardware API for System Monitoring and Control

## public for sw usage, gcp1, gcp5

Based on my analysis, **hwapi** is a **high-level, unified hardware API layer** that provides system monitoring, hardware control, and device management capabilities across Hangtu's product line.

### **What hwapi Has Done:**

#### 1. **Unified Hardware Abstraction Architecture**
- **Dynamic plugin system**: Loads hardware-specific `libhwio.so` based on product configuration
- **Configuration-driven**: Uses `/etc/hangtu/product.conf` to identify hardware platform
- **Error code standardization**: Consistent `HT_ERROR_CODE` across all functions
- **Cross-platform support**: Windows/Linux compatibility with conditional compilation

#### 2. **Core Functional Areas:**

**A. System Monitoring & Statistics**
- **CPU usage**: `/proc/stat` based calculation with caching
- **Memory usage**: `/proc/meminfo` parsing (MemTotal, MemFree, Buffers, Cached, Slab)
- **Disk usage**: `statfs()` for filesystem capacity
- **Temperature monitoring**: CPU and motherboard temperatures
- **Power monitoring**: Voltage and power state (single/dual power supplies)

**B. Hardware Control Interfaces**
- **LED control**: State setting, blinking, oneshot flashes
- **DIO control**: Digital I/O set/get operations
- **Buzzer control**: On/off functionality
- **LCD backlight**: Brightness and power control
- **Watchdog**: Initialization, feeding, cleanup

**C. Device Information & Identification**
- **Device type detection**: `HT_DEV_TYPE` enumeration
- **Serial number retrieval**: Device-specific SN reading
- **Product type identification**: `HT_PRODUCT_TYPE` from configuration
- **Architecture detection**: ARM/x86/etc. from system info
- **System information**: OS and platform details

**D. IRIG-B Time Synchronization**
- **B-code time synchronization**: Enable/disable, status checking
- **Time quality monitoring**: Leap seconds, accuracy levels, error states
- **Time zone configuration**: Manual or automatic from B-code

**E. Network & Disk Statistics**
- **Network interface monitoring**: Link state, traffic statistics (commented out)
- **Disk I/O statistics**: Read/write flows (commented out)
- **Uptime tracking**: System boot time monitoring

#### 3. **Architecture Characteristics:**

**Dynamic Library Loading**
```c
// Loads hardware-specific implementation
errorCode = ht_hwio_init();  // Dynamically loads libhwio.so
```

**Configuration-Based Initialization**
```ini
# /etc/hangtu/product.conf
[BASE_INFO]
PRODUCT_TYPE = 1      # Gateway device
ARCH = armv7l         # Architecture
SYSINFO = Linux       # OS information
```

**Layered Design**
```
Application Layer
    ↓
hwapi (This layer - unified API)
    ↓
libhwio.so (Hardware-specific implementation)
    ↓
Kernel Drivers / Hardware
```

**Error Handling Framework**
```c
typedef enum {
    HT_EC_OK = 0,
    HT_EC_PARA_ERROR,
    HT_EC_NOT_SUPPORT2,
    HT_EC_FAIL,
    HT_EC_UNKNOW_ERROR
} HT_ERROR_CODE;
```

#### 4. **MasterStats Subsystem**
- **cpu.c/h**: CPU usage statistics
- **disk.c/h**: Disk space monitoring
- **network.c/h**: Network interface statistics
- **uptime.c/h**: System uptime tracking
- **powerin.c/h**: Power input monitoring
- **utils.c/h**: Utility functions

### **Comparison with Other SDKs:**

| Aspect | hwapi | htsdk | c61/c62 SDKs | hal-dev-v3 |
|--------|-------|-------|--------------|------------|
| **Level** | **High-level API** | Mid-level SDK | Low-level SDK | **Generic HAL** |
| **Focus** | **System monitoring & control** | Hardware I/O | Product features | Core peripherals |
| **Architecture** | **Dynamic plugin** | Static library | Static library | Configuration-based |
| **Abstraction** | **Very high** (unified API) | Medium (config-driven) | Low (hardware-specific) | High (platform-agnostic) |
| **Use Case** | **Management applications** | Embedded applications | Product development | Cross-platform dev |

### **Relationship with Other SDKs:**

#### **hwapi as the Management Layer**
```c
// hwapi could use underlying SDKs through libhwio.so:
// - htsdk for DIO/AIO operations
// - c61sdk for LED control
// - c62sdk for sensor reading
```

#### **Theoretical Integration Stack:**
```
┌─────────────────────────────────┐
│      Management Applications    │  ← Uses hwapi
├─────────────────────────────────┤
│         hwapi (This)            │  ← Unified management API
├─────────────────────────────────┤
│   libhwio.so (Adapter Layer)    │  ← Could delegate to:
│                                 │     • htsdk (DIO, AIO, GPS)
│                                 │     • c61sdk (CPLD LEDs)
│                                 │     • c62sdk (sensors, audio)
├─────────────────────────────────┤
│   Hardware-Specific SDKs        │  ← htsdk, c61sdk, c62sdk
└─────────────────────────────────┘
```

### **What Makes hwapi Special:**

#### **Enterprise-Grade Management API**
- **Comprehensive error handling**: Standardized error codes across all functions
- **Resource monitoring**: CPU, memory, disk, temperature, power
- **Hardware abstraction**: Unified API across different hardware platforms
- **Time synchronization**: Industrial-grade IRIG-B support

#### **Production Deployment Features**
- **Dynamic configuration**: Runtime detection of hardware capabilities
- **Fallback mechanisms**: Graceful degradation when features unavailable
- **Performance optimization**: Cached statistics with configurable refresh rates
- **Cross-platform compatibility**: Windows/Linux support

#### **Industrial Applications**
- **Power monitoring**: Critical for UPS/battery-backed systems
- **Temperature monitoring**: Prevents overheating in enclosed environments
- **IRIG-B synchronization**: Essential for time-sensitive industrial systems
- **Hardware health**: Comprehensive system status reporting

### **Why hwapi Exists Alongside Other SDKs:**

#### **Different Abstraction Levels**
1. **hwapi**: **Management and monitoring** (what's the system status?)
2. **htsdk**: **Hardware control** (how to control the hardware?)
3. **c61/c62 SDKs**: **Product-specific features** (optimized for specific hardware)
4. **hal-dev-v3**: **Cross-platform development** (write once, run anywhere)

#### **Separation of Concerns**
- **hwapi**: System administrators, monitoring applications
- **htsdk**: Application developers, embedded programmers
- **c61/c62 SDKs**: Product developers, hardware engineers
- **hal-dev-v3**: Platform developers, framework architects

#### **Deployment Scenarios**
- **hwapi**: Used by **management software** (SNMP agents, monitoring dashboards)
- **htsdk**: Used by **application software** (custom embedded applications)
- **c61/c62 SDKs**: Used by **product firmware** (device-specific functionality)
- **hal-dev-v3**: Used by **framework developers** (creating portable applications)

### **Technical Implementation Insights:**

#### **Plugin Architecture**
```c
// hwapi delegates to hardware-specific implementation
typedef struct {
    HT_ERROR_CODE (*hwio_get_dev_type)(void);
    HT_ERROR_CODE (*hwio_get_dev_sn)(char *sn, int len);
    HT_ERROR_CODE (*hwio_single_power_state)(int powerNo, HT_POWER_STATE *p);
    // ... many more function pointers
} HWIO_FUNCTIONS;
```

#### **Configuration-Driven Feature Detection**
```c
// Only load hardware-specific library if it's a Hangtu device
if (ht_init_product_conf() == 0) {  // Hangtu device detected
    errorCode = ht_hwio_init();     // Load libhwio.so
}
```

#### **Performance Optimization**
```c
// Cached CPU usage calculation (1-second minimum interval)
if (timeNow - lastTickTime < 1000000) {
    *p = cpuRate;  // Return cached value
    return HT_EC_OK;
}
```

### **Conclusion:**

**hwapi** is the **enterprise management layer** that:
- **Provides unified hardware monitoring** across Hangtu's product line
- **Implements dynamic plugin architecture** for hardware-specific implementations
- **Serves management applications** rather than embedded applications
- **Focuses on system health, monitoring, and control**

While **theoretically redundant** with features in htsdk/c61/c62 SDKs, **hwapi serves a different purpose**:
- **htsdk**: **How to use** the hardware (application development)
- **hwapi**: **What's the status** of the hardware (system management)

This separation allows:
1. **Application developers** to use htsdk for control
2. **System administrators** to use hwapi for monitoring
3. **Both to coexist** without conflicts or dependencies

**hwapi represents the "management plane"** of Hangtu's embedded ecosystem, complementing the "data plane" provided by the other SDKs.
