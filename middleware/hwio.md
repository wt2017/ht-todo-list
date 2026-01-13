## hwio: Hardware-Specific I/O Implementation Library

## api for hw internal

Based on my analysis, **hwio** is a **hardware-specific implementation library** that provides low-level hardware access for specific Hangtu platforms, particularly the **S1100** series. It serves as the **hardware abstraction layer** that higher-level APIs like **hwapi** dynamically load and use.

### **What hwio Has Done:**

#### 1. **Hardware-Specific Implementation Layer**
- **S1100 platform support**: Complete implementation for S1100 hardware
- **Dynamic loading capability**: Designed to be loaded as `libhwio.so` by hwapi
- **Hardware abstraction**: Provides uniform API across different hardware platforms
- **Configuration-driven**: Uses `/etc/hangtu/product.conf` for device-specific settings

#### 2. **Core Hardware Features Implemented:**

**A. GPIO and Digital I/O Control**
- **LED control**: Setting/getting LED states with hardware mapping
- **Digital I/O**: DI/DO operations with configuration-based pin mapping
- **Super I/O (SIO) chip access**: Fintek chip control for advanced GPIO

**B. System Monitoring**
- **CPU temperature**: Reading CPU temperature sensors
- **Power supply monitoring**: Dual power supply status detection
- **EEPROM access**: Reading device serial numbers from EEPROM

**C. Peripheral Control**
- **Buzzer/Beep**: Audio feedback control
- **Watchdog timer**: Hardware watchdog with SIO chip integration
- **CPLD access**: Custom CPLD control via I2C

**D. Hardware Initialization**
- **SIO chip initialization**: Super I/O chip setup and configuration
- **CPLD I2C initialization**: Custom logic device communication
- **EEPROM I2C initialization**: Serial number storage access

#### 3. **Architecture Characteristics:**

**Plugin Architecture Design**
```c
// hwapi dynamically loads and calls hwio functions
typedef struct {
    unsigned short (*hwio_get_version)();
    HT_ERROR_CODE (*hwio_init)();
    HT_ERROR_CODE (*hwio_led_set)(int led, HT_LED_STATE val);
    // ... function pointer table
} HWIO_FUNCTIONS;
```

**Configuration-Based Hardware Mapping**
```ini
# /etc/hangtu/product.conf
[SECTION_ENUM]
LED1 = 1      # LED enumeration value 1 maps to LED1 section
LED2 = 2      # LED enumeration value 2 maps to LED2 section
DO1 = 101     # Digital Output 1
DI1 = 201     # Digital Input 1
```

**Hardware-Specific Implementations**
- **S1100 GPIO**: Custom GPIO control through CPLD and SIO chips
- **Fintek SIO**: Super I/O chip for advanced hardware control
- **Custom watchdog**: SIO-based watchdog instead of standard Linux watchdog

#### 4. **S1100 Platform Specifics:**

**Super I/O (SIO) Integration**
```c
sio_init();                    // Initialize Super I/O chip
sio_active();                  // Activate SIO functions
SetWdtEnable();               // Enable watchdog via SIO
```

**CPLD Communication**
```c
s1100_cpld_i2c_init();        // Initialize CPLD I2C communication
s1100_led_set_stat(led, val); // Control LEDs through CPLD
```

**EEPROM Access**
```c
eeprom_iic_init();            // Initialize EEPROM I2C
read_eeprom_sn(sn, len);      // Read serial number from EEPROM
```

### **Comparison with Other SDKs:**

| Aspect | hwio | hwapi | htsdk | Other SDKs |
|--------|------|-------|-------|------------|
| **Level** | **Lowest (hardware)** | High (management) | Mid (application) | Various |
| **Purpose** | **Hardware access** | System monitoring | Hardware control | Feature-specific |
| **Usage** | **Dynamically loaded** | Direct API calls | Static linking | Various |
| **Portability** | **Platform-specific** | Cross-platform | Multi-platform | Varies |
| **Abstraction** | **Minimal** (close to metal) | High | Medium | Varies |

### **Relationship with Other SDKs:**

#### **hwio as the Hardware Implementation Layer**
```
┌─────────────────────────────────┐
│        hwapi (Management)       │ ← Calls hwio through function pointers
├─────────────────────────────────┤
│    libhwio.so (This - hwio)     │ ← Hardware-specific implementation
├─────────────────────────────────┤
│   Hardware (S1100, CPLD, SIO)   │ ← Physical hardware
└─────────────────────────────────┘
```

#### **Complementary Roles**
- **hwio**: **How to talk to THIS hardware** (S1100-specific)
- **hwapi**: **What's the status of ANY hardware** (unified API)
- **htsdk**: **How to use common hardware features** (application-level)
- **c61/c62 SDKs**: **How to use PRODUCT hardware** (product-specific)

### **What Makes hwio Special:**

#### **True Hardware Abstraction**
- **Direct hardware access**: Bypasses standard Linux interfaces when needed
- **Vendor-specific chips**: Fintek SIO, custom CPLD, EEPROM
- **Performance optimization**: Direct register access for critical operations

#### **Production-Grade Reliability**
- **Error handling**: Comprehensive error codes and recovery
- **Resource management**: Proper initialization and cleanup
- **Logging system**: Configurable hardware operation logging

#### **Industrial Hardware Support**
- **Dual power supplies**: Monitoring and status reporting
- **Hardware watchdog**: SIO-based with configuration options
- **EEPROM storage**: Factory-programmed serial number access

### **Technical Implementation Details:**

#### **Configuration System**
```c
// Dynamic hardware mapping from configuration
hwio_SECTION_ENUM *pSectionEnum;
for (i = 0; i < g_sthwioInst.iSectionEnumNum; i++) {
    if (led != pSectionEnum->enumVal) continue;
    // Map enumeration value to hardware section
}
```

#### **Watchdog Implementation**
```c
// Custom SIO-based watchdog (not standard Linux watchdog)
SetWdtConfiguration(timeout, 1, 1, 0, 0, 2);  // SIO watchdog config
SetWdtEnable();                               // Enable via SIO
SetWdtConfiguration(timeout, -1, -1, -1, -1, -1);  // Feed watchdog
```

#### **Hardware Initialization Sequence**
1. **SIO initialization**: `sio_init()`
2. **CPLD I2C setup**: `s1100_cpld_i2c_init()`
3. **EEPROM I2C setup**: `eeprom_iic_init()`
4. **Configuration loading**: `hwio_init_product_conf()`
5. **Power monitoring**: `hwio_init_power_state()`
6. **Temperature sensing**: `hwio_init_cpu_temperature()`

### **Why hwio Exists:**

#### **Hardware Diversity Management**
- **Different platforms** require different hardware access methods
- **Vendor-specific chips** need specialized drivers
- **Performance requirements** may bypass standard Linux interfaces

#### **Plugin Architecture Benefits**
1. **Hardware independence**: Applications use hwapi, not direct hwio calls
2. **Runtime selection**: Correct hwio implementation loaded based on hardware
3. **Easy updates**: Replace libhwio.so without recompiling applications
4. **Multiple platforms**: Different hwio implementations for different hardware

#### **Separation of Concerns**
- **hwio developers**: Focus on hardware specifics
- **hwapi developers**: Focus on API design and features
- **Application developers**: Use unified hwapi without hardware knowledge

### **Limitations and Design Decisions:**

#### **Platform-Specific Nature**
- **S1100-focused**: Current implementation targets S1100 hardware
- **Hardware assumptions**: Relies on specific chips (Fintek SIO, custom CPLD)
- **Limited portability**: Not designed for other platforms without modification

#### **Feature Completeness**
- **Some features unsupported**: `hwio_mobo_temperature()`, `hwio_power_voltage()`
- **Module DIO limited**: `hwio_module_dio_set/get()` return `HT_EC_NOT_SUPPORT3`
- **Power voltage**: Returns `HT_EC_NOT_SUPPORT3` for S1100

### **Integration with hwapi:**

#### **Dynamic Loading Mechanism**
```c
// hwapi loads hwio at runtime
HT_ERROR_CODE ht_hwio_init() {
    void *handle = dlopen("libhwio.so", RTLD_LAZY);
    g_hwio->hwiofun.hwio_init = dlsym(handle, "hwio_init");
    g_hwio->hwiofun.hwio_led_set = dlsym(handle, "hwio_led_set");
    // ... load all function pointers
}
```

#### **Error Code Translation**
- **hwio error codes** mapped to **hwapi error codes**
- **Hardware-specific failures** translated to generic error types
- **Feature availability** indicated through `HT_EC_NOT_SUPPORT*` codes

### **Conclusion:**

**hwio** is the **hardware-specific implementation layer** that:
- **Provides low-level hardware access** for specific platforms (S1100)
- **Serves as a plugin** for the higher-level hwapi management layer
- **Implements vendor-specific hardware features** (Fintek SIO, custom CPLD)
- **Enables hardware abstraction** through a uniform API

While **invisible to most developers** (who use hwapi), **hwio is critical** for:
1. **Hardware support**: Enables new hardware platforms without API changes
2. **Performance optimization**: Direct hardware access for critical operations
3. **Vendor integration**: Support for specialized chips and components

**hwio represents the "device driver layer"** of Hangtu's software stack, bridging the gap between generic hardware APIs and specific hardware implementations. Its plugin architecture allows the same management applications (using hwapi) to work across different hardware platforms, each with its own hwio implementation.

This separation enables **hardware independence for applications** while maintaining **optimal performance and feature support** for each specific hardware platform.
