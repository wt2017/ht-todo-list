## htsdk: Unified Hardware SDK for Multiple Hangtu Platforms

## gcp2 v1.0, other gcp1, gcp5

Based on my analysis, **htsdk** represents a **unified, mature SDK framework** that serves as the foundation for Hangtu's embedded product line.

### **What htsdk Has Done:**

#### 1. **Comprehensive Hardware Abstraction Framework**
- **Multi-platform support**: Configuration files for C24, C31F, ENCU3000F platforms
- **Unified API layer**: Consistent interfaces across different hardware variants
- **Configuration-driven architecture**: Hardware details in config files, not code

#### 2. **Key Feature Set Implemented:**

**A. Digital I/O (DIO) System**
- **Advanced DIO management** with named I/O support
- **Bidirectional operations**: DI input reading, DO output control
- **Name-based addressing**: `dio_do_set_stat_byname("slot:power", ON)`
- **Hardware abstraction**: GPIO port mapping in configuration files

**B. Analog I/O (AIO) System**
- **Analog voltage reading** with threshold-based digital conversion
- **Dual-mode operation**: Analog measurement OR digital input (dry/wet contacts)
- **Configurable thresholds**: ON/OFF thresholds in configuration

**C. GPS Integration**
- **Complete GPS stack** with location acquisition
- **Quality/satellite tracking**
- **Serial communication abstraction** (hw/serial.c)

**D. CAN Bus Support**
- **libsocketcan-based** CAN interface
- **SocketCAN compatibility**
- **Tx/Rx examples** for communication

**E. Standard Peripherals**
- **Watchdog timer** (standard Linux interface)
- **LED control** (Linux LED subsystem)
- **Buzzer/Beep** (based on LED timing framework)

#### 3. **Architecture Characteristics:**

**Configuration-Centric Design**
```ini
# Hardware mapping in config files
[DI1]
GPIO_PORT = 21      # Physical GPIO pin
ACTIVE    = 0       # Active low

[DO1]  
GPIO_PORT = 19
ACTIVE    = 1       # Active high
DEFAULT   = off     # Initial state
```

**Device Type Detection**
```c
uint8_t machine_get_device_type(void);     // Gateway, Fire, Security, Environment
uint8_t machine_get_modular_type(void);    // NS808_1U, NS818_2U, etc.
```

**Hardware Abstraction Layer (HW)**
- `hw/serial.c`: Serial port abstraction for GPS
- Platform-specific implementations via configuration

#### 4. **Build and Deployment System**
- **Dual build systems**: CMake for full SDK, Make for individual examples
- **Static library generation**: `libhtsdk.a`
- **Example-driven development**: Each feature has dedicated demo
- **Cross-compilation support** with compiler configuration

### **Comparison with Other SDKs:**

| Aspect | htsdk | c61sdk | c62sdk | hal-dev-v3 |
|--------|-------|--------|--------|------------|
| **Scope** | **Broad, multi-platform** | Narrow (C61 LEDs) | Broad (C62 features) | **Generic, cross-platform** |
| **Architecture** | **Configuration-driven** | Direct hardware | Object-oriented | **Platform-agnostic** |
| **Target** | **Multiple Hangtu products** | Single product | Single product | **Multiple vendors** |
| **Maturity** | **Production-ready** | Production | Production | Experimental |
| **Features** | **Industrial I/O, GPS, CAN** | LED control | Sensors, audio, FFT | Core peripherals |

### **Relationship Between SDKs:**

#### **htsdk as the Foundation**
```c
// htsdk provides the core framework that could theoretically support:
// - c61sdk's LED control (via DIO/configuration)
// - c62sdk's features (via extended configuration)
// - Platform-specific optimizations
```

#### **Why Multiple SDKs Exist:**

1. **Historical Evolution**
   - **htsdk**: Original framework for industrial products (C24, C31F)
   - **c61sdk**: Specialized for C61's unique CPLD-based LEDs
   - **c62sdk**: Enhanced for C62's multimedia/processing capabilities
   - **hal-dev-v3**: Experimental cross-platform initiative

2. **Product Specialization**
   - **htsdk**: Industrial automation (DIO, AIO, GPS, CAN)
   - **c61sdk**: Embedded device with CPLD-controlled indicators
   - **c62sdk**: Multimedia/processing platform with sensors
   - **hal-dev-v3**: Generic embedded Linux development

3. **Technical Constraints**
   - **c61sdk**: Tight coupling with CPLD hardware
   - **c62sdk**: ARM NEON optimizations, specific sensors
   - **htsdk**: Configuration system for varied hardware
   - **hal-dev-v3**: Minimal dependencies for portability

### **What Makes htsdk Special:**

#### **Industrial-Grade Features**
- **GPS integration** for location-aware applications
- **CAN bus support** for automotive/industrial networks
- **Analog I/O** with configurable thresholds
- **Named I/O system** for maintainable code

#### **Production-Proven Architecture**
- **Multiple product configurations** (C24, C31F, ENCU3000F)
- **Field-tested** in various applications
- **Comprehensive documentation** and examples
- **Professional build system**

#### **Extensible Design**
```ini
# New hardware support via configuration
[NEW_PLATFORM]
DEVICE = 5
MODULAR_TYPE = 7

[NEW_FEATURE]
GPIO_PORT = 42
ACTIVE = 1
```

### **Theoretical Integration Potential:**

**htsdk could potentially subsume c61/c62 functionality through:**

1. **Extended Configuration Schema**
   ```ini
   [C61_CPLD]
   I2C_BUS = 4
   I2C_ADDR = 0x70
   LED_MAPPING = ALM:0, ENC:1, F1_GREEN:2
   
   [C62_SENSORS]
   TEMP_PATHS = /sys/class/hwmon/hwmon2/temp1_input
   BUZZER_CTRL = pwm:0
   ```

2. **Plugin Architecture**
   - Core htsdk framework
   - Platform-specific plugins (c61, c62, rpi4)
   - Feature modules (FFT, audio, advanced sensors)

3. **Unified Build System**
   - Single CMake configuration
   - Conditional compilation based on target
   - Shared utility libraries

### **Why They Remain Separate (For Now):**

1. **Development Velocity**
   - Independent teams can work on product-specific SDKs
   - Faster iteration without framework constraints

2. **Optimization Requirements**
   - c62sdk needs NEON-optimized FFT
   - c61sdk needs minimal overhead for real-time LED control
   - htsdk prioritizes configurability over raw performance

3. **Customer Requirements**
   - Different products have different API expectations
   - Backward compatibility with existing applications
   - Product-specific documentation and support

### **Conclusion:**

**htsdk** is the **mature, industrial-grade SDK framework** that:
- **Serves as the foundation** for Hangtu's product line
- **Provides comprehensive I/O capabilities** (DIO, AIO, GPS, CAN)
- **Uses configuration-driven architecture** for multi-platform support
- **Represents the "enterprise" approach** to embedded SDK design

While **theoretically capable** of covering c61/c62 features through configuration extensions, **practical considerations** (development resources, optimization needs, product timelines) keep them as separate, specialized SDKs that can evolve independently while potentially converging on htsdk's architecture in the future.
