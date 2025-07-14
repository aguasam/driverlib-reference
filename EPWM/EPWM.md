# Chapter: EPWM - Enhanced Pulse Width Modulation

# Summary

  - [Introdução](#introdução)
- [Seção 1: Funções do Driver EPWM](#seção-1-funções-do-driver-epwm)
  - [1.1 `EPWM_setEmulationMode()`](#11-epwm_setemulationmode)
  - [1.2 `EPWM_configureSignal()`](#12-epwm_configuresignal)
- [Seção 2: Funções Auxiliares (Mencionadas dentro de `EPWM_configureSignal()`)](#seção-2-funções-auxiliares-mencionadas-dentro-de-epwm_configuresignal)
  - [2.1 `EPWM_setClockPrescaler()`](#21-epwm_setclockprescaler)
  - [2.2 `EPWM_setTimeBaseCounterMode()`](#22-epwm_settimebasecountermode)
  - [2.3 `EPWM_setTimeBasePeriod()`](#23-epwm_settimebaseperiod)
  - [2.4 `EPWM_disablePhaseShiftLoad()`](#24-epwm_disablephaseshiftload)
  - [2.5 `EPWM_setPhaseShift()`](#25-epwm_setphaseshift)
  - [2.6 `EPWM_setTimeBaseCounter()`](#26-epwm_settimebasecounter)
  - [2.7 `EPWM_setCounterCompareShadowLoadMode()`](#27-epwm_setcountercompareshadowloadmode)
  - [2.8 `EPWM_setCounterCompareValue()`](#28-epwm_setcountercomparevalue)
  - [2.9 `EPWM_setActionQualifierAction()`](#29-epwm_setactionqualifieraction)
  - [2.10 `EPWM_isBaseValid()`](#210-epwm_isbasevalid)
  
## Introduction

This chapter delves into the intricacies of the **Enhanced Pulse Width Modulation (EPWM)** module driver for the Texas Instruments **DSP28004x** microcontroller. The EPWM is a powerful peripheral capable of generating precise pulse-width modulated signals, essential for applications such as motor control, power electronics, and digital-to-analog conversion. This chapter will cover the fundamental functions provided in the driver, enabling developers to effectively configure and utilize the EPWM module.

-----

## Section 1: EPWM Driver Functions

This section details the functions available in the EPWM driver, providing explanations of their purpose, parameters, and usage.

-----

### 1.1 `EPWM_setEmulationMode()`

**Purpose:** Configures the behavior of the EPWM module during emulation mode (e.g., when the debugger halts the CPU).

**Syntax:**

```c
void EPWM_setEmulationMode(uint32_t base, EPWM_EmulationMode emulationMode);
```

**Parameters:**

  * `base`: The base address of the EPWM module. Use the predefined macros (e.g., `EPWM1_BASE`, `EPWM2_BASE`) for specific EPWM instances.
  * `emulationMode`: Specifies the emulation mode. Must be one of the values from the `EPWM_EmulationMode` enum:
      * `EPWM_EMULATION_STOP_AFTER_NEXT_TB`: Stops after the next Time Base counter increment or decrement.
      * `EPWM_EMULATION_STOP_AFTER_FULL_CYCLE`: Stops when the counter completes a full cycle.
      * `EPWM_EMULATION_FREE_RUN`: Free running.

**Return Value:** None (`void` function).

**Description:** This function sets the `FREE_SOFT` bits in the `TBCTL` register, controlling how the EPWM timer behaves when the CPU is halted by the debugger. This is crucial for debugging real-time systems.

**Example:**

```c
EPWM_setEmulationMode(EPWM1_BASE, EPWM_EMULATION_STOP_AFTER_NEXT_TB);
```

This example configures EPWM1 to stop its counter after the next increment/decrement when the debugger halts the CPU.

-----

### 1.2 `EPWM_configureSignal()`

**Purpose:** Configures the primary operational parameters of an EPWM signal, including frequency, duty cycle, and counter mode.

**Syntax:**

```c
void EPWM_configureSignal(uint32_t base, const EPWM_SignalParams *signalParams);
```

**Parameters:**

  * `base`: The base address of the EPWM module (e.g., `EPWM1_BASE`).
  * `signalParams`: A pointer to a constant `EPWM_SignalParams` structure containing the settings for the EPWM signal.

**Return Value:** None (`void` function).

**Description:** This is the most comprehensive EPWM configuration function. It calculates and sets the necessary register values based on the provided parameters. It handles time-base clock prescaling, counter mode, period, compare values, and action qualifiers to generate the desired PWM waveforms on `EPWMxA` and `EPWMxB`.

**The `EPWM_SignalParams` Structure:**

```c
typedef struct
{
    float32_t             freqInHz;      //!< Desired Signal Frequency (in Hz)
    float32_t             dutyValA;      //!< Desired Duty Cycle for ePWMxA
    float32_t             dutyValB;      //!< Desired Duty Cycle for ePWMxB
    bool                  invertSignalB; //!< Inverts the ePWMxB Signal if true
    float32_t             sysClkInHz;    //!< SYSCLK Frequency (in Hz)
    EPWM_TimeBaseCountMode tbCtrMode;     //!< Time-Base Counter Mode
    EPWM_ClockDivider      tbClkDiv;      //!< Time-Base Counter Clock Divider
    EPWM_HSClockDivider    tbHSClkDiv;    //!< Time-Base Counter HS Clock Divider
} EPWM_SignalParams;
```

**Key Configuration Steps Performed by `EPWM_configureSignal()`:**

  * **Time-Base Clock Configuration:**
      * Calls `EPWM_setClockPrescaler()` to set the clock division for the time-base counter.
  * **Time-Base Counter Mode:**
      * Calls `EPWM_setTimeBaseCounterMode()` to set the counter mode (up, down, or up-down).
  * **Calculation of `TBPRD`, `CMPA`, `CMPB`:**
      * Calculates the time-base period (`TBPRD`) and compare values (`CMPA`, `CMPB`) based on the system clock, desired PWM frequency, duty cycles, and counter mode. The calculations ensure the correct frequency and duty cycle are achieved.
  * **Initial Configuration:**
      * Disables phase-shift loading, sets the phase shift to 0, and initializes the time-base counter to 0.
  * **Shadow Register Loading:**
      * Configures shadow register loading for `CMPA` and `CMPB` to occur on the time-base counter's zero event. This ensures synchronized updates of the compare values.
  * **Setting Compare Value:**
      * Calls `EPWM_setCounterCompareValue()` to set the calculated `CMPA` and `CMPB` values.
  * **Action Qualifier Configuration:**
      * Uses `EPWM_setActionQualifierAction()` to define the actions taken on the EPWM outputs (`EPWMxA` and `EPWMxB`) based on time-base counter events (zero, compare A, compare B). This is how the PWM waveform is generated. The logic changes based on the selected counter mode and whether `EPWMxB` needs to be inverted.

**Example:**

```c
EPWM_SignalParams myEPWMConfig;
myEPWMConfig.sysClkInHz = 100000000UL; // 100 MHz system clock
myEPWMConfig.freqInHz = 10000U;       // 10 kHz PWM frequency
myEPWMConfig.tbClkDiv = EPWM_CLOCK_DIVIDER_1;
myEPWMConfig.tbHSClkDiv = EPWM_HSCLOCK_DIVIDER_1;
myEPWMConfig.tbCtrMode = EPWM_COUNTER_MODE_UP_DOWN;
myEPWMConfig.dutyValA = 0.50f;        // 50% duty cycle for EPWMxA
myEPWMConfig.dutyValB = 0.50f;        // 50% duty cycle for EPWMxB
myEPWMConfig.invertSignalB = true;    // Invert EPWMxB to be complementary to EPWMxA

EPWM_configureSignal(EPWM1_BASE, &myEPWMConfig);
```

This example configures EPWM1 to generate a 10 kHz PWM signal with a 50% duty cycle on `EPWMxA` and a complementary signal on `EPWMxB`, using the up-down count mode and a 100 MHz system clock.

-----

## Section 2: Helper Functions (Mentioned within `EPWM_configureSignal`)

These functions are called internally by `EPWM_configureSignal()` to handle specific configuration tasks:

-----

### 2.1 `EPWM_setClockPrescaler()`

**Purpose:** Sets the clock dividers for the EPWM module's time-base counter. This allows controlling the frequency of the timer's counting clock, which in turn affects the maximum period and resolution of the PWM.

**Syntax:**

```c
void EPWM_setClockPrescaler(uint32_t base,
                              EPWM_ClockDivider prescaler,
                              EPWM_HSClockDivider highSpeedPrescaler);
```

**Parameters:**

  * `base`: The base address of the EPWM module (e.g., `EPWM1_BASE`).
  * `prescaler`: The main time-base clock divider. Must be one of the values from the `EPWM_ClockDivider` enum:
      * `EPWM_CLOCK_DIVIDER_1` (Divide clock by 1)
      * `EPWM_CLOCK_DIVIDER_2` (Divide clock by 2)
      * ...
      * `EPWM_CLOCK_DIVIDER_128` (Divide clock by 128)
  * `highSpeedPrescaler`: The high-speed time-base clock divider. Must be one of the values from the `EPWM_HSClockDivider` enum:
      * `EPWM_HSCLOCK_DIVIDER_1` (Divide clock by 1)
      * `EPWM_HSCLOCK_DIVIDER_2` (Divide clock by 2)
      * ...
      * `EPWM_HSCLOCK_DIVIDER_14` (Divide clock by 14)

**Return Value:** None (`void` function).

**Description:** This function configures the `CLKDIV` and `HSPCLKDIV` bits in the EPWM's `TBCTL` register. The final time-base clock frequency (`TBCLK`) is determined by the equation: `TBCLK = EPWMCLK / (highSpeedPrescaler * prescaler)`. A lower time-base clock frequency allows for longer PWM periods, while a higher frequency offers greater resolution.

**Example:**

```c
// Configure EPWM1 to divide the system clock by 2 (TBCLK)
// and the TBCLK by 4 (HSPCLK)
EPWM_setClockPrescaler(EPWM1_BASE, EPWM_CLOCK_DIVIDER_2,
                               EPWM_HSCLOCK_DIVIDER_4);
```

-----

### 2.2 `EPWM_setTimeBaseCounterMode()`

**Purpose:** Sets the counting mode of the EPWM module's time-base counter. The counting mode determines how the timer counter operates (up, down, or up-down), which is fundamental to generating different types of PWM waveforms.

**Syntax:**

```c
void EPWM_setTimeBaseCounterMode(uint32_t base, EPWM_TimeBaseCountMode counterMode);
```

**Parameters:**

  * `base`: The base address of the EPWM module (e.g., `EPWM1_BASE`).
  * `counterMode`: The desired counting mode. Must be one of the values from the `EPWM_TimeBaseCountMode` enum:
      * `EPWM_COUNTER_MODE_UP`: The counter counts from 0 up to the period (`TBPRD`).
      * `EPWM_COUNTER_MODE_DOWN`: The counter counts from the period (`TBPRD`) down to 0.
      * `EPWM_COUNTER_MODE_UP_DOWN`: The counter counts from 0 to the period (`TBPRD`) and then back to 0.
      * `EPWM_COUNTER_MODE_STOP_FREEZE`: The counter stops and freezes its current value.

**Return Value:** None (`void` function).

**Description:** This function configures the `CTRMODE` bits in the EPWM's `TBCTL` register. The choice of counting mode directly impacts how the compare events (`CMPA`, `CMPB`) are used to toggle the states of the PWM outputs (`EPWMxA`, `EPWMxB`), allowing for the creation of single-edge, dual-edge, or symmetrical waveforms.

**Example:**

```c
// Configure EPWM1 to operate in up-down count mode
EPWM_setTimeBaseCounterMode(EPWM1_BASE, EPWM_COUNTER_MODE_UP_DOWN);
```

-----

### 2.3 `EPWM_setTimeBasePeriod()`

**Purpose:** Sets the period value for the EPWM module's time-base counter. This value determines the upper limit (or upper and lower limits, depending on the counting mode) for the counter, thereby establishing the fundamental frequency of the PWM signal.

**Syntax:**

```c
void EPWM_setTimeBasePeriod(uint32_t base, uint16_t periodCount);
```

**Parameters:**

  * `base`: The base address of the EPWM module (e.g., `EPWM1_BASE`).
  * `periodCount`: The period value for the time-base counter. This value is loaded into the `TBPRD` register. The maximum value is `0xFFFF` (65535).

**Return Value:** None (`void` function).

**Description:** This function writes the provided value to the EPWM's `TBPRD` (Time Base Period) register. The time-base counter (`TBCTR`) counts from 0 up to this value (in up-count mode), or from `periodCount` down to 0 (in down-count mode), or from 0 up to `periodCount` and back to 0 (in up-down-count mode). The PWM frequency is inversely proportional to the period value and the time-base clock frequency.

**Example:**

```c
// Set the period of EPWM1 to 1000
EPWM_setTimeBasePeriod(EPWM1_BASE, 1000U);
```

-----

### 2.4 `EPWM_disablePhaseShiftLoad()`

**Purpose:** Disables the loading of the phase-shift value to the EPWM module's time-base counter. When disabled, the time-base counter is not initialized with a phase-shift value upon starting or restarting.

**Syntax:**

```c
void EPWM_disablePhaseShiftLoad(uint32_t base);
```

**Parameters:**

  * `base`: The base address of the EPWM module (e.g., `EPWM1_BASE`).

**Return Value:** None (`void` function).

**Description:** This function clears the `PHSEN` bit in the EPWM's `TBCTL` register. By disabling phase-shift loading, the time-base counter (`TBCTR`) will always start its count from 0 (or the default reset value) instead of a specific phase value. This is useful for ensuring that multiple EPWM modules start their counts synchronously or for setups where phase shifting is not required.

**Example:**

```c
// Disable phase-shift loading for EPWM1
EPWM_disablePhaseShiftLoad(EPWM1_BASE);
```

-----

### 2.5 `EPWM_setPhaseShift()`

**Purpose:** Sets the phase-shift value for the EPWM module's time-base counter. This value is used to initialize the time-base counter when phase loading is enabled, allowing for the synchronization of multiple EPWM modules with a specific delay.

**Syntax:**

```c
void EPWM_setPhaseShift(uint32_t base, uint16_t phaseCount);
```

**Parameters:**

  * `base`: The base address of the EPWM module (e.g., `EPWM1_BASE`).
  * `phaseCount`: The phase-shift value. This value is loaded into the `TBPHS` register. The maximum value is `0xFFFF` (65535).

**Return Value:** None (`void` function).

**Description:** This function writes the provided value to the EPWM's `TBPHS` (Time Base Phase) register. When phase loading is enabled (the `PHSEN` bit in `TBCTL` is set), the time-base counter (`TBCTR`) will be initialized with this value on the synchronization event (e.g., a software sync event or a sync event from another EPWM). This is fundamental for controlling the phase relationship between different PWM outputs.

**Example:**

```c
// Set a phase shift of 500 for EPWM1
EPWM_setPhaseShift(EPWM1_BASE, 500U);
```

-----

### 2.6 `EPWM_setTimeBaseCounter()`

**Purpose:** Sets the current value of the EPWM module's time-base counter. This allows initializing the counter to a specific value or modifying it at runtime.

**Syntax:**

```c
void EPWM_setTimeBaseCounter(uint32_t base, uint16_t count);
```

**Parameters:**

  * `base`: The base address of the EPWM module (e.g., `EPWM1_BASE`).
  * `count`: The value to which the time-base counter (`TBCTR`) will be set. The maximum value is `0xFFFF` (65535).

**Return Value:** None (`void` function).

**Description:** This function writes the provided `count` directly into the EPWM's `TBCTR` (Time Base Counter) register. Setting the time-base counter can be useful for:

  * Initializing the counter at a specific point in a PWM cycle.
  * Synchronizing the counter with other events or hardware modules.
  * Implementing advanced PWM control techniques that require direct manipulation of the counter.

**Example:**

```c
// Set the time-base counter of EPWM1 to 0
EPWM_setTimeBaseCounter(EPWM1_BASE, 0U);

// Set the time-base counter of EPWM2 to 250
EPWM_setTimeBaseCounter(EPWM2_BASE, 250U);
```

-----

### 2.7 `EPWM_setCounterCompareShadowLoadMode()`

**Purpose:** Configures the shadow load mode for the compare registers (`CMPA`, `CMPB`, `CMPC`, `CMPD`) of the EPWM module. Shadow loading ensures that updates to the compare values occur synchronously with timer events, preventing glitches in the PWM waveforms.

**Syntax:**

```c
void EPWM_setCounterCompareShadowLoadMode(uint32_t base,
                                          EPWM_CounterCompareModule compModule,
                                          EPWM_CounterCompareLoadMode loadMode);
```

**Parameters:**

  * `base`: The base address of the EPWM module (e.g., `EPWM1_BASE`).
  * `compModule`: The compare module to be configured. Must be one of the values from the `EPWM_CounterCompareModule` enum:
      * `EPWM_COUNTER_COMPARE_A` (Counter-compare A)
      * `EPWM_COUNTER_COMPARE_B` (Counter-compare B)
      * `EPWM_COUNTER_COMPARE_C` (Counter-compare C)
      * `EPWM_COUNTER_COMPARE_D` (Counter-compare D)
  * `loadMode`: The shadow load mode. Must be one of the values from the `EPWM_CounterCompareLoadMode` enum:
      * `EPWM_COMP_LOAD_ON_CNTR_ZERO`: Load when counter equals zero.
      * `EPWM_COMP_LOAD_ON_CNTR_PERIOD`: Load when counter equals period.
      * `EPWM_COMP_LOAD_ON_CNTR_ZERO_PERIOD`: Load when counter equals zero or period.
      * `EPWM_COMP_LOAD_FREEZE`: Freeze shadow to active load.
      * `EPWM_COMP_LOAD_ON_SYNC_CNTR_ZERO`: Load on sync or when counter equals zero.
      * `EPWM_COMP_LOAD_ON_SYNC_CNTR_PERIOD`: Load on sync or when counter equals period.
      * `EPWM_COMP_LOAD_ON_SYNC_CNTR_ZERO_PERIOD`: Load on sync or when counter equals zero or period.
      * `EPWM_COMP_LOAD_ON_SYNC_ONLY`: Load on sync only.

**Return Value:** None (`void` function).

**Description:** This function configures the shadow load control bits in the EPWM's `CMPCTL` and `CMPCTL2` registers. By using shadow loading, you ensure that changes to the compare values do not affect the PWM waveform immediately, but rather at the next specified loading event (ZERO, PRD, SYNC, or combinations thereof). This is crucial for maintaining waveform integrity and avoiding unwanted transitions.

**Example:**

```c
// Configure EPWM1's CMPA to load its shadow value on the counter's zero event
EPWM_setCounterCompareShadowLoadMode(EPWM1_BASE,
                                       EPWM_COUNTER_COMPARE_A,
                                       EPWM_COMP_LOAD_ON_CNTR_ZERO);

// Configure EPWM2's CMPB to load its shadow value on the counter's period event
EPWM_setCounterCompareShadowLoadMode(EPWM2_BASE,
                                       EPWM_COUNTER_COMPARE_B,
                                       EPWM_COMP_LOAD_ON_CNTR_PERIOD);
```

-----

### 2.8 `EPWM_setCounterCompareValue()`

**Purpose:** Sets the compare value for one of the compare registers (`CMPA`, `CMPB`, `CMPC`, or `CMPD`) of the EPWM module. These values are crucial for determining the duty cycle and edge transitions of the generated PWM signals.

**Syntax:**

```c
void EPWM_setCounterCompareValue(uint32_t base,
                                 EPWM_CounterCompareModule compModule,
                                 uint16_t compCount);
```

**Parameters:**

  * `base`: The base address of the EPWM module (e.g., `EPWM1_BASE`).
  * `compModule`: The compare module to be configured. Must be one of the values from the `EPWM_CounterCompareModule` enum:
      * `EPWM_COUNTER_COMPARE_A` (Counter-compare A)
      * `EPWM_COUNTER_COMPARE_B` (Counter-compare B)
      * `EPWM_COUNTER_COMPARE_C` (Counter-compare C)
      * `EPWM_COUNTER_COMPARE_D` (Counter-compare D)
  * `compCount`: The value to be loaded into the compare register. This value is compared with the time-base counter (`TBCTR`) to generate trigger events. The maximum value is `0xFFFF` (65535).

**Return Value:** None (`void` function).

**Description:** This function writes the provided `compCount` to the EPWM's `CMPA`, `CMPB`, `CMPC`, or `CMPD` register, depending on the selected `compModule`. The compare values are used by the action qualifiers to determine when the PWM output should change state (e.g., from high to low or vice-versa). The relationship between the compare value and the time-base period (`TBPRD`) defines the duty cycle of the PWM signal. It is important to note that if shadow loading is enabled for the compare module, the `compCount` will be written to the shadow register and will only be transferred to the active register at the next shadow load event.

**Example:**

```c
// Set the compare A value of EPWM1 to 500
EPWM_setCounterCompareValue(EPWM1_BASE, EPWM_COUNTER_COMPARE_A, 500U);

// Set the compare B value of EPWM2 to 250
EPWM_setCounterCompareValue(EPWM2_BASE, EPWM_COUNTER_COMPARE_B, 250U);
```

-----

### 2.9 `EPWM_setActionQualifierAction()`

**Purpose:** Configures the action that occurs on the `EPWMxA` or `EPWMxB` outputs in response to specific time-base counter or compare events. This is the central function for defining how the PWM waveform is generated.

**Syntax:**

```c
void EPWM_setActionQualifierAction(uint32_t base,
                                     EPWM_ActionQualifierOutputModule epwmOutput,
                                     EPWM_ActionQualifierOutput output,
                                     EPWM_ActionQualifierOutputEvent event);
```

**Parameters:**

  * `base`: The base address of the EPWM module (e.g., `EPWM1_BASE`).
  * `epwmOutput`: The EPWM output to be configured. Must be one of the values from the `EPWM_ActionQualifierOutputModule` enum:
      * `EPWM_AQ_OUTPUT_A`: For the `EPWMxA` output.
      * `EPWM_AQ_OUTPUT_B`: For the `EPWMxB` output.
  * `output`: The action to be taken on the output when the event occurs. Must be one of the values from the `EPWM_ActionQualifierOutput` enum:
      * `EPWM_AQ_OUTPUT_NO_CHANGE`: No change in the output state.
      * `EPWM_AQ_OUTPUT_LOW`: Forces the output to a low state.
      * `EPWM_AQ_OUTPUT_HIGH`: Forces the output to a high state.
      * `EPWM_AQ_OUTPUT_TOGGLE`: Toggles the current state of the output.
  * `event`: The event that triggers the action. Must be one of the values from the `EPWM_ActionQualifierOutputEvent` enum:
      * `EPWM_AQ_OUTPUT_ON_TIMEBASE_ZERO`: Action on counter reaching zero event.
      * `EPWM_AQ_OUTPUT_ON_TIMEBASE_PERIOD`: Action on counter reaching period event.
      * `EPWM_AQ_OUTPUT_ON_TIMEBASE_UP_CMPA`: Action when counter is counting up and matches `CMPA`.
      * `EPWM_AQ_OUTPUT_ON_TIMEBASE_DOWN_CMPA`: Action when counter is counting down and matches `CMPA`.
      * `EPWM_AQ_OUTPUT_ON_TIMEBASE_UP_CMPB`: Action when counter is counting up and matches `CMPB`.
      * `EPWM_AQ_OUTPUT_ON_TIMEBASE_DOWN_CMPB`: Action when counter is counting down and matches `CMPB`.
      * `EPWM_AQ_OUTPUT_ON_T1_COUNT_UP`: Action on T1 event during up-count.
      * `EPWM_AQ_OUTPUT_ON_T1_COUNT_DOWN`: Action on T1 event during down-count.
      * `EPWM_AQ_OUTPUT_ON_T2_COUNT_UP`: Action on T2 event during up-count.
      * `EPWM_AQ_OUTPUT_ON_T2_COUNT_DOWN`: Action on T2 event during down-count.

**Return Value:** None (`void` function).

**Description:** This function configures the action qualifier bits in the EPWM's `AQCTLA` and `AQCTLB` registers. It allows you to define precisely when and how the PWM outputs (`EPWMxA` and `EPWMxB`) should change state. For example, you can configure `EPWMxA` to go HIGH on the counter's ZERO event and LOW on the `UP_CMPA` event, creating a basic PWM waveform. The combination of different actions and events allows for the creation of complex and precisely controlled waveforms.

**Example:**

```c
// Configure EPWM1A to go HIGH when the counter reaches zero
EPWM_setActionQualifierAction(EPWM1_BASE,
                              EPWM_AQ_OUTPUT_A,
                              EPWM_AQ_OUTPUT_HIGH,
                              EPWM_AQ_OUTPUT_ON_TIMEBASE_ZERO);

// Configure EPWM1A to go LOW when the counter is counting up and matches CMPA
EPWM_setActionQualifierAction(EPWM1_BASE,
                              EPWM_AQ_OUTPUT_A,
                              EPWM_AQ_OUTPUT_LOW,
                              EPWM_AQ_OUTPUT_ON_TIMEBASE_UP_CMPA);

// Configure EPWM1B to toggle its state on the period event
EPWM_setActionQualifierAction(EPWM1_BASE,
                              EPWM_AQ_OUTPUT_B,
                              EPWM_AQ_OUTPUT_TOGGLE,
                              EPWM_AQ_OUTPUT_ON_TIMEBASE_PERIOD);
```

-----

### 2.10 `EPWM_isBaseValid()`

**Purpose:** Checks if the provided base address for an EPWM module is valid. This function is typically used internally by the drivers to ensure that register operations are performed on valid memory addresses, preventing invalid accesses and unexpected behavior.

**Syntax:**

```c
bool EPWM_isBaseValid(uint32_t base);
```

**Parameters:**

  * `base`: The base address of the EPWM module to be validated (e.g., `EPWM1_BASE`, `EPWM2_BASE`).

**Return Value:** `true` if the base address is valid; `false` otherwise.

**Description:** This function checks if the base address corresponds to one of the known and valid base addresses for the EPWM modules in the DSP28004x microcontroller. This is a common practice in driver libraries to ensure code robustness and safety, especially in embedded systems where invalid memory accesses can lead to critical failures.

**Example:**

```c
// Example of internal usage (as seen in other driver functions)
// ASSERT(EPWM_isBaseValid(base));

// Example of usage in application code for validation
if (EPWM_isBaseValid(EPWM3_BASE))
{
    // The EPWM3 base address is valid, proceed with configuration
    // EPWM_setEmulationMode(EPWM3_BASE, EPWM_EMULATION_FREE_RUN);
}
else
{
    // The base address is not valid, handle the error
    // Log error, return, etc.
}
```
