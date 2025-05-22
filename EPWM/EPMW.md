# Capítulo: EPWM - Modulação por Largura de Pulso Aprimorada

# Sumário
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
- [Seção 3: Considerações Importantes](#seção-3-considerações-importantes)
- [Seção 4: Submódulos Avançados do EPWM](#seção-4-submódulos-avançados-do-epwm)
  - [4.1 Submódulo PWM Chopper](#41-submódulo-pwm-chopper)
  - [4.2 Submódulo Trip-Zone](#42-submódulo-trip-zone)
  - [4.3 Submódulo Digital-Compare](#43-submódulo-digital-compare)
  - [4.4 Submódulo Event-Trigger](#44-submódulo-event-trigger)
  - [4.5 Submódulo Dead-Band](#45-submódulo-dead-band)
  - [4.6 Submódulo Minimum Dead-Band and Illegal Combination](#46-submódulo-minimum-dead-band-and-illegal-combination)
  - [4.7 Submódulo Valley Switching](#47-submódulo-valley-switching)
  - [4.8 Submódulo Global Load](#48-submódulo-global-load)
  - [4.9 Submódulo Lock Registers](#49-submódulo-lock-registers)
- [Seção 5: High Resolution PWM (HRPWM)](#seção-5-high-resolution-pwm-hrpwm)
- [Seção 6: Recursos](#seção-6-recursos)


## Introdução

Este capítulo aprofunda-se nas complexidades do driver do módulo **Enhanced Pulse Width Modulation (EPWM)** para o microcontrolador Texas Instruments **DSP28004x**. O EPWM é um periférico poderoso capaz de gerar sinais de modulação por largura de pulso precisos, essenciais para aplicações como controle de motores, eletrônica de potência e conversão digital-analógica. Este capítulo cobrirá as funções fundamentais fornecidas no driver, permitindo que os desenvolvedores configurem e utilizem efetivamente o módulo EPWM.

---

## Seção 1: Funções do Driver EPWM

Esta seção detalha as funções disponíveis no driver EPWM, fornecendo explicações sobre seu propósito, parâmetros e uso.

---

### 1.1 `EPWM_setEmulationMode()`

**Propósito:** Configura o comportamento do módulo EPWM durante o modo de emulação (por exemplo, quando o depurador pausa a CPU).

**Sintaxe:**

```c
void EPWM_setEmulationMode(uint32_t base, EPWM_EmulationMode emulationMode);
```

**Parâmetros:**

* `base`: O endereço base do módulo EPWM. Use as macros predefinidas (por exemplo, `EPWM1_BASE`, `EPWM2_BASE`) para instâncias EPWM específicas.
* `emulationMode`: Especifica o modo de emulação. Deve ser um dos valores do enum `EPWM_EmulationMode`:
    * `EPWM_EMULATION_STOP_AFTER_NEXT_TB`: Para após o próximo incremento ou decremento do contador da Base de Tempo.
    * `EPWM_EMULATION_STOP_AFTER_FULL_CYCLE`: Para quando o contador completa um ciclo inteiro.
    * `EPWM_EMULATION_FREE_RUN`: Execução livre.

**Valor de Retorno:** Nenhum (função `void`).

**Descrição:** Esta função define os bits `FREE_SOFT` no registrador `TBCTL`, controlando como o temporizador EPWM se comporta quando a CPU é pausada pelo depurador. Isso é crucial para depurar sistemas em tempo real.

**Exemplo:**

```c
EPWM_setEmulationMode(EPWM1_BASE, EPWM_EMULATION_STOP_AFTER_NEXT_TB);
```

Este exemplo configura o EPWM1 para parar seu contador após o próximo incremento/decremento quando o depurador pausa a CPU.

---

### 1.2 `EPWM_configureSignal()`

**Propósito:** Configura os parâmetros operacionais primários de um sinal EPWM, incluindo frequência, ciclo de trabalho e modo de contador.

**Sintaxe:**

```c
void EPWM_configureSignal(uint32_t base, const EPWM_SignalParams *signalParams);
```

**Parâmetros:**

* `base`: O endereço base do módulo EPWM (por exemplo, `EPWM1_BASE`).
* `signalParams`: Um ponteiro para uma estrutura `EPWM_SignalParams` constante que contém as configurações para o sinal EPWM.

**Valor de Retorno:** Nenhum (função `void`).

**Descrição:** Esta é a função de configuração EPWM mais abrangente. Ela calcula e define os valores de registro necessários com base nos parâmetros fornecidos. Ela lida com a pré-escalagem do clock da base de tempo, modo de contador, período, valores de comparação e qualificadores de ação para gerar as formas de onda PWM desejadas em `EPWMxA` e `EPWMxB`.

**A Estrutura `EPWM_SignalParams`:**

```c
typedef struct
{
    float32_t               freqInHz;      //!< Frequência do Sinal Desejada (em Hz)
    float32_t               dutyValA;      //!< Ciclo de Trabalho Desejado para ePWMxA
    float32_t               dutyValB;      //!< Ciclo de Trabalho Desejado para ePWMxB
    bool                    invertSignalB; //!< Inverte o Sinal ePWMxB se verdadeiro
    float32_t               sysClkInHz;    //!< Frequência do SYSCLK (em Hz)
    EPWM_TimeBaseCountMode  tbCtrMode;     //!< Modo de Contador da Base de Tempo
    EPWM_ClockDivider       tbClkDiv;      //!< Divisor do Clock do Contador da Base de Tempo
    EPWM_HSClockDivider     tbHSClkDiv;    //!< Divisor do Clock HS do Contador da Base de Tempo
} EPWM_SignalParams;
```

**Principais Etapas de Configuração Executadas por `EPWM_configureSignal()`:**

* **Configuração do Clock da Base de Tempo:**
    * Chama `EPWM_setClockPrescaler()` para definir a divisão do clock para o contador da base de tempo.
* **Modo de Contador da Base de Tempo:**
    * Chama `EPWM_setTimeBaseCounterMode()` para definir o modo de contador (crescente, decrescente ou crescente-decrescente).
* **Cálculo de `TBPRD`, `CMPA`, `CMPB`:**
    * Calcula o período da base de tempo (`TBPRD`) e os valores de comparação (`CMPA`, `CMPB`) com base no clock do sistema, frequência PWM desejada, ciclos de trabalho e modo de contador. Os cálculos garantem que a frequência e o ciclo de trabalho corretos sejam alcançados.
* **Configuração Inicial:**
    * Desabilita o carregamento de deslocamento de fase, define o deslocamento de fase para 0 e inicializa o contador da base de tempo para 0.
* **Carregamento do Registro Sombra:**
    * Configura o carregamento do registro sombra para `CMPA` e `CMPB` para ocorrer no evento zero do contador da base de tempo. Isso garante atualizações sincronizadas dos valores de comparação.
* **Definição do Valor de Comparação:**
    * Chama `EPWM_setCounterCompareValue()` para definir os valores calculados de `CMPA` e `CMPB`.
* **Configuração do Qualificador de Ação:**
    * Usa `EPWM_setActionQualifierAction()` para definir as ações tomadas nas saídas EPWM (`EPWMxA` e `EPWMxB`) com base nos eventos do contador da base de tempo (zero, comparação A, comparação B). É assim que a forma de onda PWM é gerada. A lógica muda com base no modo de contador selecionado e se o `EPWMxB` precisa ser invertido.

**Exemplo:**

```c
EPWM_SignalParams myEPWMConfig;
myEPWMConfig.sysClkInHz = 100000000UL; // Clock do sistema de 100 MHz
myEPWMConfig.freqInHz = 10000U;        // Frequência PWM de 10 kHz
myEPWMConfig.tbClkDiv = EPWM_CLOCK_DIVIDER_1;
myEPWMConfig.tbHSClkDiv = EPWM_HSCLOCK_DIVIDER_1;
myEPWMConfig.tbCtrMode = EPWM_COUNTER_MODE_UP_DOWN;
myEPWMConfig.dutyValA = 0.50f;         // 50% de ciclo de trabalho para EPWMxA
myEPWMConfig.dutyValB = 0.50f;         // 50% de ciclo de trabalho para EPWMxB
myEPWMConfig.invertSignalB = true;     // Inverte EPWMxB para ser complementar a EPWMxA

EPWM_configureSignal(EPWM1_BASE, &myEPWMConfig);
```

Este exemplo configura o EPWM1 para gerar um sinal PWM de 10 kHz com 50% de ciclo de trabalho em `EPWMxA` e um sinal complementar em `EPWMxB`, usando o modo de contagem crescente-decrescente e um clock de sistema de 100 MHz.

---

## Seção 2: Funções Auxiliares (Mencionadas dentro de `EPWM_configureSignal`)

Estas funções são chamadas internamente por `EPWM_configureSignal()` para lidar com tarefas de configuração específicas:

---

### 2.1 `EPWM_setClockPrescaler()`

**Propósito:** Define os divisores de clock para o contador da base de tempo do módulo EPWM. Isso permite controlar a frequência do clock de contagem do temporizador, que por sua vez afeta o período máximo e a resolução do PWM.

**Sintaxe:**

```c
void EPWM_setClockPrescaler(uint32_t base,
                            EPWM_ClockDivider prescaler,
                            EPWM_HSClockDivider highSpeedPrescaler);
```

**Parâmetros:**

* `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`).
* `prescaler`: O divisor do clock da base de tempo principal. Deve ser um dos valores do enum `EPWM_ClockDivider`:
    * `EPWM_CLOCK_DIVIDER_1` (Divide clock por 1)
    * `EPWM_CLOCK_DIVIDER_2` (Divide clock por 2)
    * ...
    * `EPWM_CLOCK_DIVIDER_128` (Divide clock por 128)
* `highSpeedPrescaler`: O divisor do clock de alta velocidade da base de tempo. Deve ser um dos valores do enum `EPWM_HSClockDivider`:
    * `EPWM_HSCLOCK_DIVIDER_1` (Divide clock por 1)
    * `EPWM_HSCLOCK_DIVIDER_2` (Divide clock por 2)
    * ...
    * `EPWM_HSCLOCK_DIVIDER_14` (Divide clock por 14)

**Valor de Retorno:** Nenhum (função `void`).

**Descrição:** Esta função configura os bits `CLKDIV` e `HSPCLKDIV` no registrador `TBCTL` do EPWM. A frequência final do clock da base de tempo (`TBCLK`) é determinada pela equação: `TBCLK = EPWMCLK / (highSpeedPrescaler * prescaler)`. Uma frequência de clock da base de tempo mais baixa permite períodos de PWM mais longos, enquanto uma frequência mais alta oferece maior resolução.

**Exemplo:**

```c
// Configura o EPWM1 para dividir o clock do sistema por 2 (TBCLK)
// e o TBCLK por 4 (HSPCLK)
EPWM_setClockPrescaler(EPWM1_BASE, EPWM_CLOCK_DIVIDER_2,
                       EPWM_HSCLOCK_DIVIDER_4);
```

---

### 2.2 `EPWM_setTimeBaseCounterMode()`

**Propósito:** Define o modo de contagem do contador da base de tempo do módulo EPWM. O modo de contagem determina como o contador do temporizador opera (crescente, decrescente ou crescente-decrescente), o que é fundamental para a geração de diferentes tipos de formas de onda PWM.

**Sintaxe:**

```c
void EPWM_setTimeBaseCounterMode(uint32_t base, EPWM_TimeBaseCountMode counterMode);
```

**Parâmetros:**

* `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`).
* `counterMode`: O modo de contagem desejado. Deve ser um dos valores do enum `EPWM_TimeBaseCountMode`:
    * `EPWM_COUNTER_MODE_UP`: O contador conta de 0 até o período (`TBPRD`).
    * `EPWM_COUNTER_MODE_DOWN`: O contador conta do período (`TBPRD`) até 0.
    * `EPWM_COUNTER_MODE_UP_DOWN`: O contador conta de 0 até o período (`TBPRD`) e depois de volta para 0.
    * `EPWM_COUNTER_MODE_STOP_FREEZE`: O contador para e congela seu valor atual.

**Valor de Retorno:** Nenhum (função `void`).

**Descrição:** Esta função configura os bits `CTRMODE` no registrador `TBCTL` do EPWM. A escolha do modo de contagem impacta diretamente como os eventos de comparação (`CMPA`, `CMPB`) são usados para alternar os estados das saídas PWM (`EPWMxA`, `EPWMxB`), permitindo a criação de formas de onda de borda única, borda dupla ou simétricas.

**Exemplo:**

```c
// Configura o EPWM1 para operar no modo de contagem crescente-decrescente
EPWM_setTimeBaseCounterMode(EPWM1_BASE, EPWM_COUNTER_MODE_UP_DOWN);
```

---

### 2.3 `EPWM_setTimeBasePeriod()`

**Propósito:** Define o valor do período para o contador da base de tempo do módulo EPWM. Este valor determina o limite superior (ou limite superior e inferior, dependendo do modo de contagem) para o contador, estabelecendo assim a frequência fundamental do sinal PWM.

**Sintaxe:**

```c
void EPWM_setTimeBasePeriod(uint32_t base, uint16_t periodCount);
```

**Parâmetros:**

* `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`).
* `periodCount`: O valor do período para o contador da base de tempo. Este valor é carregado no registrador `TBPRD`. O valor máximo é `0xFFFF` (65535).

**Valor de Retorno:** Nenhum (função `void`).

**Descrição:** Esta função escreve o valor fornecido no registrador `TBPRD` (Time Base Period) do EPWM. O contador da base de tempo (`TBCTR`) conta de 0 até este valor (no modo crescente), ou de `periodCount` até 0 (no modo decrescente), ou de 0 até `periodCount` e de volta para 0 (no modo crescente-decrescente). A frequência do PWM é inversamente proporcional ao valor do período e à frequência do clock da base de tempo.

**Exemplo:**

```c
// Define o período do EPWM1 para 1000
EPWM_setTimeBasePeriod(EPWM1_BASE, 1000U);
```

---

### 2.4 `EPWM_disablePhaseShiftLoad()`

**Propósito:** Desabilita o carregamento do valor de deslocamento de fase para o contador da base de tempo do módulo EPWM. Quando desabilitado, o contador da base de tempo não é inicializado com um valor de deslocamento de fase ao iniciar ou reiniciar.

**Sintaxe:**

```c
void EPWM_disablePhaseShiftLoad(uint32_t base);
```

**Parâmetros:**

* `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`).

**Valor de Retorno:** Nenhum (função `void`).

**Descrição:** Esta função limpa o bit `PHSEN` no registrador `TBCTL` do EPWM. Ao desabilitar o carregamento de deslocamento de fase, o contador da base de tempo (`TBCTR`) sempre iniciará sua contagem a partir de 0 (ou do valor de reset padrão) em vez de um valor de fase específico. Isso é útil para garantir que vários módulos EPWM iniciem suas contagens de forma síncrona ou para configurações onde o deslocamento de fase não é necessário.

**Exemplo:**

```c
// Desabilita o carregamento de deslocamento de fase para o EPWM1
EPWM_disablePhaseShiftLoad(EPWM1_BASE);
```

---

### 2.5 `EPWM_setPhaseShift()`

**Propósito:** Define o valor de deslocamento de fase para o contador da base de tempo do módulo EPWM. Este valor é usado para inicializar o contador da base de tempo quando o carregamento de fase está habilitado, permitindo a sincronização de múltiplos módulos EPWM com um atraso específico.

**Sintaxe:**

```c
void EPWM_setPhaseShift(uint32_t base, uint16_t phaseCount);
```

**Parâmetros:**

* `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`).
* `phaseCount`: O valor de deslocamento de fase. Este valor é carregado no registrador `TBPHS`. O valor máximo é `0xFFFF` (65535).

**Valor de Retorno:** Nenhum (função `void`).

**Descrição:** Esta função escreve o valor fornecido no registrador `TBPHS` (Time Base Phase) do EPWM. Quando o carregamento de fase está habilitado (o bit `PHSEN` no `TBCTL` está definido), o contador da base de tempo (`TBCTR`) será inicializado com este valor no evento de sincronização (por exemplo, um evento de sincronização de software ou um evento de sincronização de outro EPWM). Isso é fundamental para controlar a relação de fase entre diferentes saídas PWM.

**Exemplo:**

```c
// Define um deslocamento de fase de 500 para o EPWM1
EPWM_setPhaseShift(EPWM1_BASE, 500U);
```

---

### 2.6 `EPWM_setTimeBaseCounter()`

**Propósito:** Define o valor atual do contador da base de tempo do módulo EPWM. Isso permite inicializar o contador para um valor específico ou modificá-lo em tempo de execução.

**Sintaxe:**

```c
void EPWM_setTimeBaseCounter(uint32_t base, uint16_t count);
```

**Parâmetros:**

* `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`).
* `count`: O valor para o qual o contador da base de tempo (`TBCTR`) será definido. O valor máximo é `0xFFFF` (65535).

**Valor de Retorno:** Nenhum (função `void`).

**Descrição:** Esta função escreve o `count` fornecido diretamente no registrador `TBCTR` (Time Base Counter) do EPWM. Definir o contador da base de tempo pode ser útil para:

* Inicializar o contador em um ponto específico de um ciclo PWM.
* Sincronizar o contador com outros eventos ou módulos de hardware.
* Implementar técnicas avançadas de controle de PWM que exigem manipulação direta do contador.

**Exemplo:**

```c
// Define o contador da base de tempo do EPWM1 para 0
EPWM_setTimeBaseCounter(EPWM1_BASE, 0U);

// Define o contador da base de tempo do EPWM2 para 250
EPWM_setTimeBaseCounter(EPWM2_BASE, 250U);
```

---

### 2.7 `EPWM_setCounterCompareShadowLoadMode()`

**Propósito:** Configura o modo de carregamento sombra para os registradores de comparação (`CMPA`, `CMPB`, `CMPC`, `CMPD`) do módulo EPWM. O carregamento sombra garante que as atualizações dos valores de comparação ocorram de forma síncrona com os eventos do temporizador, evitando glitches nas formas de onda PWM.

**Sintaxe:**

```c
void EPWM_setCounterCompareShadowLoadMode(uint32_t base,
                                        EPWM_CounterCompareModule compModule,
                                        EPWM_CounterCompareLoadMode loadMode);
```

**Parâmetros:**

* `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`).
* `compModule`: O módulo de comparação a ser configurado. Deve ser um dos valores do enum `EPWM_CounterCompareModule`:
    * `EPWM_COUNTER_COMPARE_A` (Contador comparador A)
    * `EPWM_COUNTER_COMPARE_B` (Contador comparador B)
    * `EPWM_COUNTER_COMPARE_C` (Contador comparador C)
    * `EPWM_COUNTER_COMPARE_D` (Contador comparador D)
* `loadMode`: O modo de carregamento sombra. Deve ser um dos valores do enum `EPWM_CounterCompareLoadMode`:
    * `EPWM_COMP_LOAD_ON_CNTR_ZERO`: Carrega quando o contador é igual a zero.
    * `EPWM_COMP_LOAD_ON_CNTR_PERIOD`: Carrega quando o contador é igual ao período.
    * `EPWM_COMP_LOAD_ON_CNTR_ZERO_PERIOD`: Carrega quando o contador é igual a zero ou período.
    * `EPWM_COMP_LOAD_FREEZE`: Congela o carregamento sombra para ativo.
    * `EPWM_COMP_LOAD_ON_SYNC_CNTR_ZERO`: Carrega na sincronização ou quando o contador é igual a zero.
    * `EPWM_COMP_LOAD_ON_SYNC_CNTR_PERIOD`: Carrega na sincronização ou quando o contador é igual ao período.
    * `EPWM_COMP_LOAD_ON_SYNC_CNTR_ZERO_PERIOD`: Carrega na sincronização ou quando o contador é igual a zero ou período.
    * `EPWM_COMP_LOAD_ON_SYNC_ONLY`: Carrega apenas na sincronização.

**Valor de Retorno:** Nenhum (função `void`).

**Descrição:** Esta função configura os bits de controle de carregamento sombra nos registradores `CMPCTL` e `CMPCTL2` do EPWM. Ao utilizar o carregamento sombra, você garante que as alterações nos valores de comparação não afetem a forma de onda PWM imediatamente, mas sim no próximo evento de carregamento especificado (ZERO, PRD, SYNC ou suas combinações). Isso é crucial para manter a integridade da forma de onda e evitar transições indesejadas.

**Exemplo:**

```c
// Configura o CMPA do EPWM1 para carregar seu valor sombra no evento zero do contador
EPWM_setCounterCompareShadowLoadMode(EPWM1_BASE,
                                     EPWM_COUNTER_COMPARE_A,
                                     EPWM_COMP_LOAD_ON_CNTR_ZERO);

// Configura o CMPB do EPWM2 para carregar seu valor sombra no evento de período do contador
EPWM_setCounterCompareShadowLoadMode(EPWM2_BASE,
                                     EPWM_COUNTER_COMPARE_B,
                                     EPWM_COMP_LOAD_ON_CNTR_PERIOD);
```

---

### 2.8 `EPWM_setCounterCompareValue()`

**Propósito:** Define o valor de comparação para um dos registradores de comparação (`CMPA`, `CMPB`, `CMPC` ou `CMPD`) do módulo EPWM. Esses valores são cruciais para determinar o ciclo de trabalho e as transições de borda dos sinais PWM gerados.

**Sintaxe:**

```c
void EPWM_setCounterCompareValue(uint32_t base,
                                 EPWM_CounterCompareModule compModule,
                                 uint16_t compCount);
```

**Parâmetros:**

* `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`).
* `compModule`: O módulo de comparação a ser configurado. Deve ser um dos valores do enum `EPWM_CounterCompareModule`:
    * `EPWM_COUNTER_COMPARE_A` (Contador comparador A)
    * `EPWM_COUNTER_COMPARE_B` (Contador comparador B)
    * `EPWM_COUNTER_COMPARE_C` (Contador comparador C)
    * `EPWM_COUNTER_COMPARE_D` (Contador comparador D)
* `compCount`: O valor a ser carregado no registrador de comparação. Este valor é comparado com o contador da base de tempo (`TBCTR`) para gerar eventos de acionamento. O valor máximo é `0xFFFF` (65535).

**Valor de Retorno:** Nenhum (função `void`).

**Descrição:** Esta função escreve o `compCount` fornecido no registrador `CMPA`, `CMPB`, `CMPC` ou `CMPD` do EPWM, dependendo do `compModule` selecionado. Os valores de comparação são usados pelos qualificadores de ação para determinar quando a saída PWM deve mudar de estado (por exemplo, de alta para baixa ou vice-versa). A relação entre o valor de comparação e o período da base de tempo (`TBPRD`) define o ciclo de trabalho do sinal PWM. É importante notar que, se o carregamento sombra estiver habilitado para o módulo de comparação, o `compCount` será escrito no registrador sombra e só será transferido para o registrador ativo no próximo evento de carregamento sombra.

**Exemplo:**

```c
// Define o valor de comparação A do EPWM1 para 500
EPWM_setCounterCompareValue(EPWM1_BASE, EPWM_COUNTER_COMPARE_A, 500U);

// Define o valor de comparação B do EPWM2 para 250
EPWM_setCounterCompareValue(EPWM2_BASE, EPWM_COUNTER_COMPARE_B, 250U);
```

---

### 2.9 `EPWM_setActionQualifierAction()`

**Propósito:** Configura a ação que ocorre nas saídas `EPWMxA` ou `EPWMxB` em resposta a eventos específicos do contador da base de tempo ou de comparação. Esta é a função central para definir como a forma de onda PWM é gerada.

**Sintaxe:**

```c
void EPWM_setActionQualifierAction(uint32_t base,
                                   EPWM_ActionQualifierOutputModule epwmOutput,
                                   EPWM_ActionQualifierOutput output,
                                   EPWM_ActionQualifierOutputEvent event);
```

**Parâmetros:**

* `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`).
* `epwmOutput`: A saída EPWM a ser configurada. Deve ser um dos valores do enum `EPWM_ActionQualifierOutputModule`:
    * `EPWM_AQ_OUTPUT_A`: Para a saída `EPWMxA`.
    * `EPWM_AQ_OUTPUT_B`: Para a saída `EPWMxB`.
* `output`: A ação a ser tomada na saída quando o evento ocorrer. Deve ser um dos valores do enum `EPWM_ActionQualifierOutput`:
    * `EPWM_AQ_OUTPUT_NO_CHANGE`: Nenhuma mudança no estado da saída.
    * `EPWM_AQ_OUTPUT_LOW`: Força a saída para o estado baixo.
    * `EPWM_AQ_OUTPUT_HIGH`: Força a saída para o estado alto.
    * `EPWM_AQ_OUTPUT_TOGGLE`: Inverte o estado atual da saída.
* `event`: O evento que aciona a ação. Deve ser um dos valores do enum `EPWM_ActionQualifierOutputEvent`:
    * `EPWM_AQ_OUTPUT_ON_TIMEBASE_ZERO`: Ação no evento de contador atingir zero.
    * `EPWM_AQ_OUTPUT_ON_TIMEBASE_PERIOD`: Ação no evento de contador atingir o período.
    * `EPWM_AQ_OUTPUT_ON_TIMEBASE_UP_CMPA`: Ação quando o contador está crescendo e atinge `CMPA`.
    * `EPWM_AQ_OUTPUT_ON_TIMEBASE_DOWN_CMPA`: Ação quando o contador está decrescendo e atinge `CMPA`.
    * `EPWM_AQ_OUTPUT_ON_TIMEBASE_UP_CMPB`: Ação quando o contador está crescendo e atinge `CMPB`.
    * `EPWM_AQ_OUTPUT_ON_TIMEBASE_DOWN_CMPB`: Ação quando o contador está decrescendo e atinge `CMPB`.
    * `EPWM_AQ_OUTPUT_ON_T1_COUNT_UP`: Ação no evento T1 durante a contagem crescente.
    * `EPWM_AQ_OUTPUT_ON_T1_COUNT_DOWN`: Ação no evento T1 durante a contagem decrescente.
    * `EPWM_AQ_OUTPUT_ON_T2_COUNT_UP`: Ação no evento T2 durante a contagem crescente.
    * `EPWM_AQ_OUTPUT_ON_T2_COUNT_DOWN`: Ação no evento T2 durante a contagem decrescente.

**Valor de Retorno:** Nenhum (função `void`).

**Descrição:** Esta função configura os bits de ação qualificador nos registradores `AQCTLA` e `AQCTLB` do EPWM. Ela permite que você defina com precisão quando e como as saídas PWM (`EPWMxA` e `EPWMxB`) devem mudar de estado. Por exemplo, você pode configurar `EPWMxA` para ir para ALTO no evento ZERO do contador e para BAIXO no evento `UP_CMPA`, criando uma forma de onda PWM básica. A combinação de diferentes ações e eventos permite a criação de formas de onda complexas e controladas com precisão.

**Exemplo:**

```c
// Configura EPWM1A para ir para ALTO quando o contador atinge zero
EPWM_setActionQualifierAction(EPWM1_BASE,
                              EPWM_AQ_OUTPUT_A,
                              EPWM_AQ_OUTPUT_HIGH,
                              EPWM_AQ_OUTPUT_ON_TIMEBASE_ZERO);

// Configura EPWM1A para ir para BAIXO quando o contador está crescendo e atinge CMPA
EPWM_setActionQualifierAction(EPWM1_BASE,
                              EPWM_AQ_OUTPUT_A,
                              EPWM_AQ_OUTPUT_LOW,
                              EPWM_AQ_OUTPUT_ON_TIMEBASE_UP_CMPA);

// Configura EPWM1B para alternar seu estado no evento de período
EPWM_setActionQualifierAction(EPWM1_BASE,
                              EPWM_AQ_OUTPUT_B,
                              EPWM_AQ_OUTPUT_TOGGLE,
                              EPWM_AQ_OUTPUT_ON_TIMEBASE_PERIOD);
```

---

### 2.10 `EPWM_isBaseValid()`

**Propósito:** Verifica se o endereço base fornecido para um módulo EPWM é válido. Esta função é tipicamente usada internamente pelos drivers para garantir que as operações de registro sejam realizadas em endereços de memória válidos, prevenindo acessos inválidos e comportamentos inesperados.

**Sintaxe:**

```c
bool EPWM_isBaseValid(uint32_t base);
```

**Parâmetros:**

* `base`: O endereço base do módulo EPWM a ser validado (e.g., `EPWM1_BASE`, `EPWM2_BASE`).

**Valor de Retorno:** `true` se o endereço base for válido; `false` caso contrário.

**Descrição:** Esta função verifica se o endereço base corresponde a um dos endereços base conhecidos e válidos para os módulos EPWM no microcontrolador DSP28004x. Isso é uma prática comum em bibliotecas de drivers para garantir a robustez e a segurança do código, especialmente em sistemas embarcados onde acessos de memória inválidos podem levar a falhas críticas.

**Exemplo:**

```c
// Exemplo de uso interno (como visto em outras funções do driver)
// ASSERT(EPWM_isBaseValid(base));

// Exemplo de uso em código de aplicação para validação
if (EPWM_isBaseValid(EPWM3_BASE))
{
    // O endereço base do EPWM3 é válido, prossiga com a configuração
    // EPWM_setEmulationMode(EPWM3_BASE, EPWM_EMULATION_FREE_RUN);
}
else
{
    // O endereço base não é válido, tratar o erro
    // Logar erro, retornar, etc.
}
```

---

## Seção 3: Considerações Importantes

* **Clocking:** A geração precisa de PWM depende de um clock de sistema estável e preciso. Preste atenção à configuração da árvore de clock do seu DSP28004x.
* **Shadowing de Registradores:** O módulo EPWM frequentemente usa registradores sombra para atualizar valores críticos (como valores de comparação) de forma sincronizada, evitando falhas na forma de onda de saída. As funções do driver lidam com isso adequadamente.
* **Qualificadores de Ação:** Os qualificadores de ação são o coração da geração de PWM. Entender como eles controlam a saída EPWM com base nos eventos do contador é essencial para aplicações EPWM avançadas.
* **Tratamento de Erros:** As macros `ASSERT()` no código do driver são para depuração. Em um ambiente de produção, você pode querer adicionar um tratamento de erros mais robusto.

---

## Seção 4: Submódulos Avançados do EPWM

Esta seção descreve os submódulos mais avançados do EPWM, que oferecem funcionalidades especializadas para aplicações de controle de alta performance.

---

### 4.1 Submódulo PWM Chopper

**Propósito:** O submódulo PWM Chopper é utilizado principalmente com gate drives baseados em transformadores de pulso para controlar dispositivos de comutação de potência. Ele modula o sinal PWM principal com um sinal de portadora de alta frequência para criar uma saída "picada" (chopped).

**Visão Geral:** Este submódulo utiliza um sinal de portadora de alta frequência em conjunto com a forma de onda PWM gerada pelos submódulos anteriores (base de tempo, contador-comparador, qualificador de ação e gerador de tempo morto).

**Funcionamento:** Um sinal de portadora de alta frequência (`CHPFREQ`) é aplicado em uma operação lógica AND com os sinais EPWM base para produzir as saídas "picadas" finais. Adicionalmente, este submódulo oferece uma opção para incluir um pulso one-shot (`OSHT`) maior antes dos pulsos de sustentação.

**Diagrama de Forma de Onda (Conceitual):**

[A imagem acima ilustra como um sinal PWM base é modulado por uma portadora de alta frequência para criar a saída "picada".]

**Recursos Principais:**

* Modulação de alta frequência dos sinais PWM.
* Capacidade de adicionar um pulso inicial one-shot de largura maior.
* Ideal para aplicações que exigem isolamento galvânico ou redução de perdas em transformadores de pulso.

**Funções da Driverlib:**

* **`EPWM_enableChopper(uint32_t base)`**
    * **Propósito:** Habilita o submódulo Chopper do ePWM.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Esta função define o bit `CHPEN` no registrador `PCCTL` para ativar o submódulo Chopper.

* **`EPWM_disableChopper(uint32_t base)`**
    * **Propósito:** Desabilita o submódulo Chopper do ePWM.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Esta função limpa o bit `CHPEN` no registrador `PCCTL` para desativar o submódulo Chopper.

* **`EPWM_setChopperDutyCycle(uint32_t base, uint16_t dutyCycleCount)`**
    * **Propósito:** Define o ciclo de trabalho do clock de chopping.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `dutyCycleCount`: Contagem do ciclo de trabalho do clock de chopping. Deve ser menor que 7.
    * **Retorno:** Nenhum.
    * **Descrição:** Esta função configura o ciclo de trabalho do clock de chopping. O valor `dutyCycleCount` é convertido para o ciclo de trabalho real do chopper pela equação: ciclo de trabalho do chopper = (`dutyCycleCount` + 1) / 8.

* **`EPWM_setChopperFreq(uint32_t base, uint16_t freqDiv)`**
    * **Propósito:** Define o escalonador de frequência do clock de chopping.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `freqDiv`: Divisor de frequência do clock de chopping. Deve ser menor que 8.
    * **Retorno:** Nenhum.
    * **Descrição:** Esta função define o escalonador para a frequência do clock de chopping. A frequência do clock de chopping é alterada com base na equação: frequência do clock do chopper = `SYSCLKOUT` / (1 + `freqDiv`).

* **`EPWM_setChopperFirstPulseWidth(uint32_t base, uint16_t firstPulseWidth)`**
    * **Propósito:** Define a largura do primeiro pulso da forma de onda de saída do chopper.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `firstPulseWidth`: Largura do primeiro pulso. Deve ser menor que `0x10` (16).
    * **Retorno:** Nenhum.
    * **Descrição:** Esta função define a largura do primeiro pulso da forma de onda de saída do chopper. O valor da largura do primeiro pulso em segundos é dado pela equação: largura do primeiro pulso = 1 / (((`firstPulseWidth` + 1) * `SYSCLKOUT`)/8).

---

### 4.2 Submódulo Trip-Zone

**Propósito:** O submódulo Trip-Zone e Digital-Compare fornecem um mecanismo de proteção para salvaguardar os pinos de saída de anormalidades, como sobretensão, sobrecorrente e aumento excessivo de temperatura.

**Mecanismo de Proteção:** A Trip-Zone utiliza um mecanismo de lógica rápida e independente do clock para lidar rapidamente com condições de falha, forçando as saídas `EPWMxA` e `EPWMxB` para um estado seguro e configurável (como alto, baixo ou alta impedância). Devido à sua velocidade e conexão de hardware, a Trip-Zone pode ser usada quando as interrupções (software ISR) podem não ser rápidas o suficiente para proteger o hardware em resposta a condições de sobrecorrente ou curtos-circuitos.

**Tipos de Trip:**

* **One-shot (OSHT) trips:** Para grandes curtos-circuitos ou condições de sobrecorrente. Uma vez ativada, a saída PWM permanece no estado seguro até ser rearmada por software.
* **Cycle-by-cycle (CBC) trips:** Para operação de limitação de corrente. A ação de trip é aplicada a cada ciclo do PWM, permitindo uma recuperação automática se a condição de falha for transitória.

**Sinais de Trip-Zone:** Os sinais de Trip-Zone (`TZ1`-`TZ6`) podem vir de diversas fontes:

* `TZ1`-`TZ3`: Externamente, de qualquer pino GPIO. Um sinal GPIO específico pode ser roteado para a Trip-Zone do EPWM usando o Módulo X-BAR de ENTRADA.
* `TZ4`: Internamente, de um sinal de erro eQEP invertido.
* `TZ5`: Internamente, de falha do clock do sistema.
* `TZ6`: Internamente, de uma saída de parada de emulação da CPU.

Além disso, inúmeras fontes de sinal de Trip-Zone podem ser geradas pelo subsistema Digital-Compare.

**Proteção do Drive de Potência:** É um recurso de segurança para a operação segura de sistemas como conversores de potência e drives de motor. Pode ser usado para informar o programa de monitoramento sobre anormalidades como sobretensão, sobrecorrente e aumento excessivo de temperatura. Se a interrupção de proteção do drive de potência não for mascarada, os pinos de saída PWM serão colocados em um estado seguro imediatamente após o pino ser acionado para baixo. Uma interrupção também será gerada.

**EPWM X-BAR:** O EPWM X-BAR é usado para rotear vários sinais internos e externos para os módulos ePWM. Oito sinais de trip do ePWM X-BAR são roteados para todos os módulos ePWM. A arquitetura do EPWM X-BAR pode selecionar um único sinal ou combinar logicamente (OR) até 32 sinais.

**Funções da Driverlib (Exemplos Conceituais):**

* **`EPWM_enableTripZoneSignals(uint32_t base, uint16_t tzSignal)`**
    * **Propósito:** Habilita os sinais de Trip Zone especificados como fonte para o módulo Trip Zone.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `tzSignal`: Sinal(is) de Trip Zone a ser(em) habilitado(s). Pode ser um OR lógico de valores como `EPWM_TZ_SIGNAL_CBC1`, `EPWM_TZ_SIGNAL_OSHT1`, etc.
    * **Retorno:** Nenhum.
    * **Descrição:** Esta função define os bits correspondentes no registrador `TZSEL` para habilitar as fontes de trip especificadas.

* **`EPWM_disableTripZoneSignals(uint32_t base, uint16_t tzSignal)`**
    * **Propósito:** Desabilita os sinais de Trip Zone especificados como fonte para o módulo Trip Zone.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `tzSignal`: Sinal(is) de Trip Zone a ser(em) desabilitado(s). Pode ser um OR lógico de valores como `EPWM_TZ_SIGNAL_CBC1`, `EPWM_TZ_SIGNAL_OSHT1`, etc.
    * **Retorno:** Nenhum.
    * **Descrição:** Esta função limpa os bits correspondentes no registrador `TZSEL` para desabilitar as fontes de trip especificadas.

* **`EPWM_setTripZoneDigitalCompareEventCondition(uint32_t base, EPWM_TripZoneDigitalCompareOutput dcType, EPWM_TripZoneDigitalCompareOutputEvent dcEvent)`**
    * **Propósito:** Define as condições de comparação digital que causam um evento de Trip Zone.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `dcType`: Tipo de saída de Comparação Digital (e.g., `EPWM_TZ_DC_OUTPUT_A1`, `EPWM_TZ_DC_OUTPUT_A2`).
        * `dcEvent`: Evento de saída de Comparação Digital que causa o Trip Zone (e.g., `EPWM_TZ_EVENT_DCXH_LOW`, `EPWM_TZ_EVENT_DCXH_HIGH`).
    * **Retorno:** Nenhum.
    * **Descrição:** Esta função configura o registrador `TZDCSEL` para associar eventos de comparação digital a condições de Trip Zone.

* **`EPWM_enableTripZoneAdvAction(uint32_t base)`**
    * **Propósito:** Habilita as ações avançadas dos eventos de Trip Zone.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Esta função define o bit `ETZE` no registrador `TZCTL2` para habilitar recursos avançados que combinam eventos de trip zone com a direção do contador.

* **`EPWM_disableTripZoneAdvAction(uint32_t base)`**
    * **Propósito:** Desabilita as ações avançadas dos eventos de Trip Zone.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Esta função limpa o bit `ETZE` no registrador `TZCTL2`.

* **`EPWM_setTripZoneAction(uint32_t base, EPWM_TripZoneEvent tzEvent, EPWM_TripZoneAction tzAction)`**
    * **Propósito:** Define a ação de Trip Zone a ser tomada quando um evento de Trip Zone ocorre.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `tzEvent`: Tipo de evento de Trip Zone (e.g., `EPWM_TZ_ACTION_EVENT_TZA`, `EPWM_TZ_ACTION_EVENT_DCAEVT1`).
        * `tzAction`: Ação de Trip Zone (e.g., `EPWM_TZ_ACTION_HIGH_Z`, `EPWM_TZ_ACTION_HIGH`, `EPWM_TZ_ACTION_LOW`, `EPWM_TZ_ACTION_DISABLE`).
    * **Retorno:** Nenhum.
    * **Descrição:** Esta função configura o registrador `TZCTL` para definir como as saídas PWM reagem a eventos de Trip Zone.

* **`EPWM_setTripZoneAdvAction(uint32_t base, EPWM_TripZoneAdvancedEvent tzAdvEvent, EPWM_TripZoneAdvancedAction tzAdvAction)`**
    * **Propósito:** Define a ação avançada de Trip Zone a ser tomada quando um evento avançado de Trip Zone ocorre.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `tzAdvEvent`: Tipo de evento avançado de Trip Zone (e.g., `EPWM_TZ_ADV_ACTION_EVENT_TZA_U`, `EPWM_TZ_ADV_ACTION_EVENT_TZB_D`).
        * `tzAdvAction`: Ação avançada de Trip Zone (e.g., `EPWM_TZ_ADV_ACTION_HIGH_Z`, `EPWM_TZ_ADV_ACTION_TOGGLE`).
    * **Retorno:** Nenhum.
    * **Descrição:** Esta função configura o registrador `TZCTL2` para definir ações de Trip Zone que levam em consideração a direção do contador.

* **`EPWM_setTripZoneAdvDigitalCompareActionA(uint32_t base, EPWM_TripZoneAdvDigitalCompareEvent tzAdvDCEvent, EPWM_TripZoneAdvancedAction tzAdvDCAction)`**
    * **Propósito:** Define a ação avançada de Comparação Digital de Trip Zone a ser tomada na saída `ePWMA`.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `tzAdvDCEvent`: Tipo de evento de Comparação Digital de Trip Zone (e.g., `EPWM_TZ_ADV_ACTION_EVENT_DCxEVT1_U`).
        * `tzAdvDCAction`: Ação avançada de Comparação Digital de Trip Zone.
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o registrador `TZCTLDCA` para definir ações avançadas de Trip Zone para `ePWMA` baseadas em eventos de Comparação Digital.

* **`EPWM_setTripZoneAdvDigitalCompareActionB(uint32_t base, EPWM_TripZoneAdvDigitalCompareEvent tzAdvDCEvent, EPWM_TripZoneAdvancedAction tzAdvDCAction)`**
    * **Propósito:** Define a ação avançada de Comparação Digital de Trip Zone a ser tomada na saída `ePWMB`.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `tzAdvDCEvent`: Tipo de evento de Comparação Digital de Trip Zone.
        * `tzAdvDCAction`: Ação avançada de Comparação Digital de Trip Zone.
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o registrador `TZCTLDCB` para definir ações avançadas de Trip Zone para `ePWMB` baseadas em eventos de Comparação Digital.

* **`EPWM_enableTripZoneInterrupt(uint32_t base, uint16_t tzInterrupt)`**
    * **Propósito:** Habilita as interrupções de Trip Zone.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `tzInterrupt`: Interrupção(ões) de Trip Zone a ser(em) habilitada(s) (e.g., `EPWM_TZ_INTERRUPT_CBC`, `EPWM_TZ_INTERRUPT_OST`).
    * **Retorno:** Nenhum.
    * **Descrição:** Habilita interrupções específicas de Trip Zone no registrador `TZEINT`.

* **`EPWM_disableTripZoneInterrupt(uint32_t base, uint16_t tzInterrupt)`**
    * **Propósito:** Desabilita as interrupções de Trip Zone.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `tzInterrupt`: Interrupção(ões) de Trip Zone a ser(em) desabilitada(s).
    * **Retorno:** Nenhum.
    * **Descrição:** Desabilita interrupções específicas de Trip Zone no registrador `TZEINT`.

* **`EPWM_getTripZoneFlagStatus(uint32_t base)`**
    * **Propósito:** Retorna o status da flag de Trip Zone.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** O status da flag de Trip Zone (e.g., `EPWM_TZ_INTERRUPT`, `EPWM_TZ_FLAG_CBC`).
    * **Descrição:** Lê o registrador `TZFLG` para obter o status das flags de Trip Zone.

* **`EPWM_getCycleByCycleTripZoneFlagStatus(uint32_t base)`**
    * **Propósito:** Retorna o status específico da flag de Trip Zone Ciclo a Ciclo.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** O status da flag CBC (e.g., `EPWM_TZ_CBC_FLAG_1`).
    * **Descrição:** Lê o registrador `TZCBCFLG` para obter o status das flags de Trip Zone Ciclo a Ciclo.

* **`EPWM_getOneShotTripZoneFlagStatus(uint32_t base)`**
    * **Propósito:** Retorna o status específico da flag de Trip Zone One Shot.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** O status da flag OST (e.g., `EPWM_TZ_OST_FLAG_OST1`).
    * **Descrição:** Lê o registrador `TZOSTFLG` para obter o status das flags de Trip Zone One Shot.

* **`EPWM_selectCycleByCycleTripZoneClearEvent(uint32_t base, EPWM_CycleByCycleTripZoneClearMode clearEvent)`**
    * **Propósito:** Define o evento que limpa automaticamente o latch CBC (Ciclo a Ciclo).
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `clearEvent`: Modo de limpeza do pulso CBC (e.g., `EPWM_TZ_CBC_PULSE_CLR_CNTR_ZERO`).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o registrador `TZCLR` para definir o evento que reinicia o latch CBC.

* **`EPWM_clearTripZoneFlag(uint32_t base, uint16_t tzFlags)`**
    * **Propósito:** Limpa as flags de Trip Zone.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `tzFlags`: Flags de Trip Zone a serem limpas (e.g., `EPWM_TZ_INTERRUPT`, `EPWM_TZ_FLAG_CBC`).
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa as flags de Trip Zone no registrador `TZCLR`.

* **`EPWM_clearCycleByCycleTripZoneFlag(uint32_t base, uint16_t tzCBCFlags)`**
    * **Propósito:** Limpa a flag específica de Trip Zone Ciclo a Ciclo.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `tzCBCFlags`: Flags CBC a serem limpas (e.g., `EPWM_TZ_CBC_FLAG_1`).
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa as flags CBC no registrador `TZCBCCLR`.

* **`EPWM_clearOneShotTripZoneFlag(uint32_t base, uint16_t tzOSTFlags)`**
    * **Propósito:** Limpa a flag específica de Trip Zone One Shot.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `tzOSTFlags`: Flags OST a serem limpas (e.g., `EPWM_TZ_OST_FLAG_OST1`).
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa as flags OST no registrador `TZOSTCLR`.

* **`EPWM_forceTripZoneEvent(uint32_t base, uint16_t tzForceEvent)`**
    * **Propósito:** Força um evento de Trip Zone.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `tzForceEvent`: Evento de Trip Zone a ser forçado (e.g., `EPWM_TZ_FORCE_EVENT_CBC`, `EPWM_TZ_FORCE_EVENT_OST`).
    * **Retorno:** Nenhum.
    * **Descrição:** Força um evento de Trip Zone no registrador `TZFRC`.

---

### 4.3 Submódulo Digital-Compare

**Propósito:** O submódulo Digital-Compare, assim como a Trip-Zone, também pode ajudar a proteger os pinos de saída de anormalidades. Este submódulo compara sinais externos ao módulo EPWM, como um sinal dos comparadores analógicos CMPSS, para gerar diretamente eventos ou ações de comparação digital PWM.

**Eventos e Ações:** Esses eventos ou ações de comparação digital podem ser usados pelos submódulos Trip-Zone, Base de Tempo e Event-Trigger para realizar o seguinte:

* Acionar um Trip no EPWM.
* Gerar uma interrupção de Trip.
* Sincronizar o EPWM.
* Gerar um pulso de Início de Conversão (SOC) para o ADC.

**Funcionalidade de Blanking:** Você também pode usar a funcionalidade de 'Blanking' do submódulo Digital-Compare para desabilitar temporariamente as ações PWM por um período de tempo, a fim de eliminar efeitos de ruído.

**Diagrama de Submódulo (Conceitual):**

[A imagem ilustra a estrutura e as conexões do submódulo Digital-Compare.]

**Eventos Digital-Compare:** Um evento de comparação digital é gerado quando uma ou mais de suas entradas selecionadas estão em estado alto ou baixo. As entradas para o submódulo Digital-Compare são provenientes de:

* INPUT X-BAR
* EPWM X-BAR
* Pinos de entrada da Trip-Zone

**Configuração de Eventos:** Para usar os eventos de comparação digital, o usuário seleciona a entrada para cada sinal (`DCAH`, `DCAL`, `DCBH`, `DCBL`) e o estado de cada sinal que acionará cada comparação. Cada canal EPWM (A e B) usa suas entradas `DCyH/L` correspondentes (onde `y` = A ou B).

**Funções da Driverlib (Exemplos Conceituais):**

* **`EPWM_selectDigitalCompareTripInput(uint32_t base, EPWM_DigitalCompareTripInput tripSource, EPWM_DigitalCompareType dcType)`**
    * **Propósito:** Define a entrada de trip para o Digital Compare (DC).
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `tripSource`: Fonte de trip (e.g., `EPWM_DC_TRIP_TRIPIN1`, `EPWM_DC_TRIP_COMBINATION`).
        * `dcType`: Tipo de Comparação Digital (e.g., `EPWM_DC_TYPE_DCAH`, `EPWM_DC_TYPE_DCAL`).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o registrador `DCTRIPSEL` para selecionar a fonte de trip para as entradas do Digital Compare.

* **`EPWM_enableDigitalCompareBlankingWindow(uint32_t base)`**
    * **Propósito:** Habilita a janela de blanking do filtro DC.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Define o bit `BLANKE` no registrador `DCFCTL` para habilitar a janela de blanking.

* **`EPWM_disableDigitalCompareBlankingWindow(uint32_t base)`**
    * **Propósito:** Desabilita a janela de blanking do filtro DC.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa o bit `BLANKE` no registrador `DCFCTL`.

* **`EPWM_enableDigitalCompareWindowInverseMode(uint32_t base)`**
    * **Propósito:** Habilita o modo inverso da janela de Comparação Digital.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Define o bit `BLANKINV` no registrador `DCFCTL` para inverter a janela de blanking.

* **`EPWM_disableDigitalCompareWindowInverseMode(uint32_t base)`**
    * **Propósito:** Desabilita o modo inverso da janela de Comparação Digital.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa o bit `BLANKINV` no registrador `DCFCTL`.

* **`EPWM_setDigitalCompareBlankingEvent(uint32_t base, EPWM_DigitalCompareBlankingPulse blankingPulse)`**
    * **Propósito:** Define o pulso de entrada que inicia a janela de blanking do Digital Compare.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `blankingPulse`: Pulso que inicia a janela de blanking (e.g., `EPWM_DC_WINDOW_START_TBCTR_PERIOD`).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o registrador `DCFCTL` para selecionar o evento que inicia a janela de blanking.

* **`EPWM_setDigitalCompareFilterInput(uint32_t base, EPWM_DigitalCompareFilterInput filterInput)`**
    * **Propósito:** Define a fonte do sinal de entrada que será filtrada pelo módulo Digital Compare.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `filterInput`: Fonte do sinal (e.g., `EPWM_DC_WINDOW_SOURCE_DCAEVT1`).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o registrador `DCFCTL` para selecionar a fonte de entrada do filtro.

* **`EPWM_enableDigitalCompareEdgeFilter(uint32_t base)`**
    * **Propósito:** Habilita o filtro de borda do Digital Compare.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Define o bit `EDGEFILTSEL` no registrador `DCFCTL`.

* **`EPWM_disableDigitalCompareEdgeFilter(uint32_t base)`**
    * **Propósito:** Desabilita o filtro de borda do Digital Compare.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa o bit `EDGEFILTSEL` no registrador `DCFCTL`.

* **`EPWM_setDigitalCompareEdgeFilterMode(uint32_t base, EPWM_DigitalCompareEdgeFilterMode edgeMode)`**
    * **Propósito:** Define o modo do filtro de borda do Digital Compare.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `edgeMode`: Modo do filtro de borda (e.g., `EPWM_DC_EDGEFILT_MODE_RISING`, `EPWM_DC_EDGEFILT_MODE_BOTH`).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o registrador `DCFCTL` para definir o modo de detecção de borda.

* **`EPWM_setDigitalCompareEdgeFilterEdgeCount(uint32_t base, uint16_t edgeCount)`**
    * **Propósito:** Define a contagem de bordas do filtro de borda do Digital Compare para gerar eventos.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `edgeCount`: Contagem de bordas (0-7, e.g., `EPWM_DC_EDGEFILT_EDGECNT_0`).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o registrador `DCFCTL` para definir quantas bordas são necessárias para gerar um evento.

* **`EPWM_getDigitalCompareEdgeFilterEdgeCount(uint32_t base)`**
    * **Propósito:** Retorna a contagem de bordas configurada para o filtro de borda do Digital Compare.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** A contagem de bordas (0-7).
    * **Descrição:** Lê o registrador `DCFCTL` para retornar a contagem de bordas configurada.

* **`EPWM_getDigitalCompareEdgeFilterEdgeStatus(uint32_t base)`**
    * **Propósito:** Retorna o status da contagem de bordas capturadas pelo filtro de borda do Digital Compare.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** A contagem de bordas capturadas (0-7).
    * **Descrição:** Lê o registrador `DCFCTL` para retornar a contagem de bordas capturadas.

* **`EPWM_setDigitalCompareWindowOffset(uint32_t base, uint16_t windowOffsetCount)`**
    * **Propósito:** Define o offset da janela de filtro do Digital Compare.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `windowOffsetCount`: Comprimento do offset da janela de blanking em contagens de `TBCLK`.
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o registrador `DCFOFFSET` para definir o offset da janela de blanking.

* **`EPWM_setDigitalCompareWindowLength(uint32_t base, uint16_t windowLengthCount)`**
    * **Propósito:** Define o comprimento da janela de filtro do Digital Compare.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `windowLengthCount`: Comprimento da janela de blanking em contagens de `TBCLK`.
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o registrador `DCFWINDOW` para definir o comprimento da janela de blanking.

* **`EPWM_getDigitalCompareBlankingWindowOffsetCount(uint32_t base)`**
    * **Propósito:** Retorna a contagem do offset da janela de blanking do filtro DC.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** A contagem do offset.
    * **Descrição:** Retorna o valor do registrador `DCFOFFSETCNT`.

* **`EPWM_getDigitalCompareBlankingWindowLengthCount(uint32_t base)`**
    * **Propósito:** Retorna a contagem do comprimento da janela de blanking do filtro DC.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** A contagem do comprimento.
    * **Descrição:** Retorna o valor do registrador `DCFWINDOWCNT`.

* **`EPWM_setDigitalCompareEventSource(uint32_t base, EPWM_DigitalCompareModule dcModule, EPWM_DigitalCompareEvent dcEvent, EPWM_DigitalCompareEventSource dcEventSource)`**
    * **Propósito:** Define a fonte do evento para o módulo Digital Compare.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `dcModule`: Módulo Digital Compare (e.g., `EPWM_DC_MODULE_A`).
        * `dcEvent`: Número do evento Digital Compare (e.g., `EPWM_DC_EVENT_1`).
        * `dcEventSource`: Fonte do evento (e.g., `EPWM_DC_EVENT_SOURCE_FILT_SIGNAL`).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura os registradores `DCACTL` ou `DCBCTL` para selecionar a fonte do evento de comparação digital.

* **`EPWM_setDigitalCompareEventSyncMode(uint32_t base, EPWM_DigitalCompareModule dcModule, EPWM_DigitalCompareEvent dcEvent, EPWM_DigitalCompareSyncMode syncMode)`**
    * **Propósito:** Define o modo de sincronização da entrada do evento Digital Compare.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `dcModule`: Módulo Digital Compare.
        * `dcEvent`: Número do evento Digital Compare.
        * `syncMode`: Modo de sincronização (e.g., `EPWM_DC_EVENT_INPUT_SYNCED`).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura os registradores `DCACTL` ou `DCBCTL` para definir se o sinal de entrada DC é sincronizado com o `TBCLK`.

* **`EPWM_enableDigitalCompareADCTrigger(uint32_t base, EPWM_DigitalCompareModule dcModule)`**
    * **Propósito:** Habilita o evento 1 do Digital Compare para gerar um Início de Conversão (SOC) para o ADC.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `dcModule`: Módulo Digital Compare.
    * **Retorno:** Nenhum.
    * **Descrição:** Define o bit `EVT1SOCE` no registrador `DCACTL` ou `DCBCTL`.

* **`EPWM_disableDigitalCompareADCTrigger(uint32_t base, EPWM_DigitalCompareModule dcModule)`**
    * **Propósito:** Desabilita o evento 1 do Digital Compare de gerar um Início de Conversão (SOC) para o ADC.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `dcModule`: Módulo Digital Compare.
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa o bit `EVT1SOCE` no registrador `DCACTL` ou `DCBCTL`.

* **`EPWM_enableDigitalCompareSyncEvent(uint32_t base, EPWM_DigitalCompareModule dcModule)`**
    * **Propósito:** Habilita o evento 1 do Digital Compare para gerar um pulso de sincronização de saída.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `dcModule`: Módulo Digital Compare.
    * **Retorno:** Nenhum.
    * **Descrição:** Define o bit `EVT1SYNCE` no registrador `DCACTL` ou `DCBCTL`.

* **`EPWM_disableDigitalCompareSyncEvent(uint32_t base, EPWM_DigitalCompareModule dcModule)`**
    * **Propósito:** Desabilita o evento 1 do Digital Compare de gerar um pulso de sincronização de saída.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `dcModule`: Módulo Digital Compare.
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa o bit `EVT1SYNCE` no registrador `DCACTL` ou `DCBCTL`.

* **`EPWM_enableDigitalCompareCounterCapture(uint32_t base)`**
    * **Propósito:** Habilita o controlador de captura do contador da Base de Tempo.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Define o bit `CAPE` no registrador `DCCAPCTL`.

* **`EPWM_disableDigitalCompareCounterCapture(uint32_t base)`**
    * **Propósito:** Desabilita o controlador de captura do contador da Base de Tempo.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa o bit `CAPE` no registrador `DCCAPCTL`.

* **`EPWM_setDigitalCompareCounterShadowMode(uint32_t base, bool enableShadowMode)`**
    * **Propósito:** Define o modo de leitura do valor do contador da Base de Tempo.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `enableShadowMode`: `true` para ler do registrador sombra, `false` para ler do registrador ativo.
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o bit `SHDWMODE` no registrador `DCCAPCTL`.

* **`EPWM_getDigitalCompareCaptureStatus(uint32_t base)`**
    * **Propósito:** Retorna o status do evento de captura DC.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** `true` se um evento de captura DC ocorreu, `false` caso contrário.
    * **Descrição:** Retorna o status do bit `CAPSTS` no registrador `DCCAPCTL`.

* **`EPWM_clearDigitalCompareCaptureStatusFlag(uint32_t base)`**
    * **Propósito:** Limpa a flag de status latched de captura DC.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa o bit `CAPSTS` no registrador `DCCAPCTL`.

* **`EPWM_configureDigitalCompareCounterCaptureMode(uint32_t base, bool disableClearMode)`**
    * **Propósito:** Configura o modo de operação de captura do contador DC.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `disableClearMode`: `true` para desabilitar a limpeza automática, `false` para habilitar.
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o bit `CAPMODE` no registrador `DCCAPCTL`.

* **`EPWM_getDigitalCompareCaptureCount(uint32_t base)`**
    * **Propósito:** Retorna o valor de captura do contador da Base de Tempo DC.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** O valor da contagem de captura.
    * **Descrição:** Retorna o valor do registrador `DCCAP`.

* **`EPWM_enableDigitalCompareTripCombinationInput(uint32_t base, uint16_t tripInput, EPWM_DigitalCompareType dcType)`**
    * **Propósito:** Habilita a entrada combinacional de TRIP do DC.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `tripInput`: Número do Trip (e.g., `EPWM_DC_COMBINATIONAL_TRIPIN1`).
        * `dcType`: Tipo de Comparação Digital.
    * **Retorno:** Nenhum.
    * **Descrição:** Habilita uma entrada de trip específica para a combinação de Digital Compare.

* **`EPWM_disableDigitalCompareTripCombinationInput(uint32_t base, uint16_t tripInput, EPWM_DigitalCompareType dcType)`**
    * **Propósito:** Desabilita a entrada combinacional de TRIP do DC.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `tripInput`: Número do Trip.
        * `dcType`: Tipo de Comparação Digital.
    * **Retorno:** Nenhum.
    * **Descrição:** Desabilita uma entrada de trip específica para a combinação de Digital Compare.

---

### 4.4 Submódulo Event-Trigger

**Propósito:** O submódulo Event-Trigger pode usar os eventos gerados pelos submódulos Base de Tempo, Contador-Comparador e Digital-Compare para acionar dois tipos de ações:

* Gerar uma interrupção para a CPU.
* Gerar um pulso de Início de Conversão (SOC) para o ADC.

**Lógica de Pré-escalagem:** O submódulo Event-Trigger também incorpora lógica de pré-escalagem para emitir uma solicitação de interrupção ou um SOC do ADC a cada evento ou a cada até o décimo quinto evento.

**Diagrama de Interrupções e SOC (Conceitual):**

[A imagem ilustra como os eventos são usados para gerar interrupções e SOCs do ADC.]

**Eventos de Acionamento:** Esses acionadores de evento podem ocorrer em vários momentos configuráveis, como quando o contador da base de tempo é igual a zero, período, zero ou período, ou na correspondência de contagem crescente ou decrescente de um valor de comparação (`CMPx`). O subsistema Digital-Compare também pode ser usado para gerar um SOC do ADC com base em um ou mais eventos de comparação. Note que os acionadores de contagem crescente e decrescente são independentes e separados.

**Funções da Driverlib:**

* **`EPWM_enableInterrupt(uint32_t base)`**
    * **Propósito:** Habilita a interrupção do ePWM.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Define o bit `INTEN` no registrador `ETSEL` para habilitar a interrupção do ePWM.

* **`EPWM_disableInterrupt(uint32_t base)`**
    * **Propósito:** Desabilita a interrupção do ePWM.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa o bit `INTEN` no registrador `ETSEL`.

* **`EPWM_setInterruptSource(uint32_t base, uint16_t interruptSource)`**
    * **Propósito:** Define a fonte da interrupção do ePWM.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `interruptSource`: Fonte da interrupção (e.g., `EPWM_INT_TBCTR_ZERO`, `EPWM_INT_TBCTR_U_CMPA`).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o registrador `ETSEL` para selecionar o evento que gera a interrupção.

* **`EPWM_setInterruptEventCount(uint32_t base, uint16_t eventCount)`**
    * **Propósito:** Define a contagem de eventos de interrupção do ePWM.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `eventCount`: Contagem de eventos para a escala da interrupção (0-15).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o registrador `ETPS` e `ETINTPS` para definir quantos eventos devem ocorrer antes que uma interrupção seja emitida.

* **`EPWM_getEventTriggerInterruptStatus(uint32_t base)`**
    * **Propósito:** Retorna o status da interrupção.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** `true` se a interrupção do ePWM foi gerada, `false` caso contrário.
    * **Descrição:** Retorna o status do bit `INT` no registrador `ETFLG`.

* **`EPWM_clearEventTriggerInterruptFlag(uint32_t base)`**
    * **Propósito:** Limpa a flag de interrupção do ePWM.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa o bit `INT` no registrador `ETCLR`.

* **`EPWM_enableInterruptEventCountInit(uint32_t base)`**
    * **Propósito:** Habilita o carregamento pré-interrupção da contagem.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Habilita o contador de interrupção do ePWM para ser pré-carregado com um valor de contagem.

* **`EPWM_disableInterruptEventCountInit(uint32_t base)`**
    * **Propósito:** Desabilita o carregamento da contagem de interrupção.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Desabilita o contador de interrupção do ePWM de ser carregado com um valor de contagem pré-interrupção.

* **`EPWM_forceInterruptEventCountInit(uint32_t base)`**
    * **Propósito:** Força um carregamento de contador de evento pré-interrupção por software.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Força o contador de interrupção do ePWM a ser carregado com o conteúdo definido por `EPWM_setInterruptEventCountInitValue()`.

* **`EPWM_setInterruptEventCountInitValue(uint32_t base, uint16_t eventCount)`**
    * **Propósito:** Define os valores de contagem de evento de interrupção.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `eventCount`: Valor de contagem de interrupção do ePWM (0-15).
    * **Retorno:** Nenhum.
    * **Descrição:** Define o valor de pré-interrupção a ser carregado no contador de interrupção.

* **`EPWM_getInterruptEventCount(uint32_t base)`**
    * **Propósito:** Retorna a contagem de eventos de interrupção.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** A contagem de eventos de interrupção que ocorreram.
    * **Descrição:** Retorna o valor do contador de eventos de interrupção.

* **`EPWM_forceEventTriggerInterrupt(uint32_t base)`**
    * **Propósito:** Força uma interrupção do ePWM.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Define o bit `INT` no registrador `ETFRC` para forçar uma interrupção.

* **`EPWM_enableADCTrigger(uint32_t base, EPWM_ADCStartOfConversionType adcSOCType)`**
    * **Propósito:** Habilita o evento SOC do ADC.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `adcSOCType`: Tipo de SOC do ADC (e.g., `EPWM_SOC_A`, `EPWM_SOC_B`).
    * **Retorno:** Nenhum.
    * **Descrição:** Habilita o módulo ePWM para acionar um evento SOC do ADC.

* **`EPWM_disableADCTrigger(uint32_t base, EPWM_ADCStartOfConversionType adcSOCType)`**
    * **Propósito:** Desabilita o evento SOC do ADC.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `adcSOCType`: Tipo de SOC do ADC.
    * **Retorno:** Nenhum.
    * **Descrição:** Desabilita o módulo ePWM de acionar um evento SOC do ADC.

* **`EPWM_setADCTriggerSource(uint32_t base, EPWM_ADCStartOfConversionType adcSOCType, EPWM_ADCStartOfConversionSource socSource)`**
    * **Propósito:** Define a fonte do SOC do ADC do ePWM.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `adcSOCType`: Tipo de SOC do ADC.
        * `socSource`: Fonte do SOC (e.g., `EPWM_SOC_TBCTR_ZERO`, `EPWM_SOC_TBCTR_U_CMPA`).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o registrador `ETSEL` para selecionar a fonte do evento SOC do ADC.

* **`EPWM_setADCTriggerEventPrescale(uint32_t base, EPWM_ADCStartOfConversionType adcSOCType, uint16_t preScaleCount)`**
    * **Propósito:** Define a contagem de eventos SOC do ADC do ePWM.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `adcSOCType`: Tipo de SOC do ADC.
        * `preScaleCount`: Número de contagem de eventos (1-15).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o registrador `ETSOCPS` para definir quantos eventos devem ocorrer antes que um SOC seja emitido.

* **`EPWM_getADCTriggerFlagStatus(uint32_t base, EPWM_ADCStartOfConversionType adcSOCType)`**
    * **Propósito:** Retorna o status do evento SOC.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `adcSOCType`: Tipo de SOC do ADC.
    * **Retorno:** `true` se o SOC do `adcSOCType` selecionado foi gerado, `false` caso contrário.
    * **Descrição:** Retorna o status da flag SOC no registrador `ETFLG`.

* **`EPWM_clearADCTriggerFlag(uint32_t base, EPWM_ADCStartOfConversionType adcSOCType)`**
    * **Propósito:** Limpa a flag SOC do ePWM.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `adcSOCType`: Tipo de SOC do ADC.
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa a flag SOC no registrador `ETCLR`.

* **`EPWM_enableADCTriggerEventCountInit(uint32_t base, EPWM_ADCStartOfConversionType adcSOCType)`**
    * **Propósito:** Habilita o carregamento pré-SOC da contagem de eventos.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `adcSOCType`: Tipo de SOC do ADC.
    * **Retorno:** Nenhum.
    * **Descrição:** Habilita o contador de eventos SOC do ePWM para ser carregado antes de um evento SOC.

* **`EPWM_disableADCTriggerEventCountInit(uint32_t base, EPWM_ADCStartOfConversionType adcSOCType)`**
    * **Propósito:** Desabilita o carregamento pré-SOC da contagem de eventos.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `adcSOCType`: Tipo de SOC do ADC.
    * **Retorno:** Nenhum.
    * **Descrição:** Desabilita o contador de eventos SOC do ePWM de ser carregado antes de um evento SOC.

* **`EPWM_forceADCTriggerEventCountInit(uint32_t base, EPWM_ADCStartOfConversionType adcSOCType)`**
    * **Propósito:** Força um carregamento de contador de evento pré-SOC por software.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `adcSOCType`: Tipo de SOC do ADC.
    * **Retorno:** Nenhum.
    * **Descrição:** Força o contador SOC do ePWM a ser carregado com o conteúdo definido por `EPWM_setADCTriggerEventCountInitValue()`.

* **`EPWM_setADCTriggerEventCountInitValue(uint32_t base, EPWM_ADCStartOfConversionType adcSOCType, uint16_t eventCount)`**
    * **Propósito:** Define os valores de contagem do Trigger do ADC.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `adcSOCType`: Tipo de SOC do ADC.
        * `eventCount`: Valor de contagem do Trigger do ADC (0-15).
    * **Retorno:** Nenhum.
    * **Descrição:** Define os valores de contagem do Trigger do ADC.

* **`EPWM_getADCTriggerEventCount(uint32_t base, EPWM_ADCStartOfConversionType adcSOCType)`**
    * **Propósito:** Retorna a contagem de eventos SOC.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `adcSOCType`: Tipo de SOC do ADC.
    * **Retorno:** A contagem de eventos SOC que ocorreram.
    * **Descrição:** Retorna o valor do contador de eventos SOC.

* **`EPWM_forceADCTrigger(uint32_t base, EPWM_ADCStartOfConversionType adcSOCType)`**
    * **Propósito:** Força um evento SOC.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `adcSOCType`: Tipo de SOC do ADC.
    * **Retorno:** Nenhum.
    * **Descrição:** Força um evento SOC do ePWM.

---

### 4.5 Submódulo Dead-Band

**Propósito:** O submódulo Dead-Band fornece uma maneira de atrasar a comutação do PWM (de alto para baixo ou de baixo para alto). Ao atrasar a transição dos sinais PWM, você pode dar tempo para que os gates do PWM desliguem e evitar um curto-circuito.

**Visão Geral:** O submódulo Dead-Band suporta atrasos de borda de subida e borda de descida programáveis independentemente, com várias opções para gerar as saídas de sinal apropriadas em `EPWMxA` e `EPWMxB`.

**Por que usar Dead-Band?**

As saídas EPWM comutam ligando e desligando os gates dos transistores. No entanto, os gates dos transistores ligam mais rápido do que desligam! Se dois gates estiverem ligados ao mesmo tempo (mesmo que momentaneamente), isso produz um caminho do trilho de alimentação para o terra e resulta em um curto-circuito. O submódulo Dead-Band pode aliviar esse problema.

* O controle de tempo morto (`dead-band`) fornece um meio conveniente de combater problemas de *shoot-through* de corrente em um conversor de potência. O *shoot-through* ocorre quando ambos os *gates* superior e inferior na mesma fase de um conversor de potência estão abertos simultaneamente (ambos os *gates* "ligados"). Essa condição *curto-circuita* a fonte de alimentação e resulta em um grande consumo de corrente. Problemas de *shoot-through* ocorrem porque os transistores abrem mais rápido do que fecham, e porque os *gates* do conversor de potência do lado alto e do lado baixo são tipicamente comutados de forma complementar. Embora a duração do caminho de corrente de *shoot-through* seja finita durante o ciclo PWM (ou seja, o *gate* de fechamento eventualmente desligará), mesmo breves períodos de uma condição de curto-circuito podem produzir aquecimento excessivo e sobrecarregar o conversor de potência e a fonte de alimentação.

**Prevenindo Curtos-Circuitos:** Existem duas abordagens básicas para controlar o shoot-through:

1.  **Modificar os transistores:**
    * O tempo de abertura do gate do transistor deve ser aumentado para que ele (ligeiramente) exceda o tempo de fechamento. Uma maneira de conseguir isso é adicionando um conjunto de componentes passivos, como resistores e diodos em série com o gate do transistor.
    * O resistor atua para limitar a taxa de aumento de corrente em direção ao gate durante a abertura do transistor, aumentando assim o tempo de abertura. Ao fechar o transistor, no entanto, a corrente flui sem impedimentos do gate através do diodo de bypass e o tempo de fechamento, portanto, não é afetado. Embora essa abordagem passiva ofereça uma solução barata que é independente do microprocessador de controle, é imprecisa, os parâmetros do componente devem ser adaptados individualmente ao conversor de potência e não pode se adaptar às condições variáveis do sistema.
2.  **Modificar os sinais de gate PWM que controlam os transistores:**
    * Esta abordagem separa as transições em sinais PWM complementares com um período de tempo fixo. Isso é chamado de dead-band. Embora seja possível realizar a implementação de dead-band por software, os MCUs C2000 oferecem hardware on-chip para essa finalidade que não requer overhead adicional da CPU. Em comparação com a abordagem passiva, o dead-band oferece um controle mais preciso dos requisitos de tempo de gate. Além disso, o tempo morto é tipicamente especificado com uma única variável de programa que é facilmente alterada para diferentes conversores de potência ou adaptada online.

**Funções da Driverlib:**

* **`EPWM_setDeadBandOutputSwapMode(uint32_t base, EPWM_DeadBandOutput output, bool enableSwapMode)`**
    * **Propósito:** Define o modo de troca de saída do sinal Dead Band.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `output`: Saída Dead Band do ePWM (e.g., `EPWM_DB_OUTPUT_A`, `EPWM_DB_OUTPUT_B`).
        * `enableSwapMode`: `true` para habilitar a troca de saída, `false` para desabilitar.
    * **Retorno:** Nenhum.
    * **Descrição:** Esta função configura o bit `OUTSWAP` no registrador `DBCTL` para trocar a saída do sinal.

* **`EPWM_setDeadBandDelayMode(uint32_t base, EPWM_DeadBandDelayMode delayMode, bool enableDelayMode)`**
    * **Propósito:** Define o modo de atraso do sinal Dead Band.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `delayMode`: Tipo de atraso Dead Band (e.g., `EPWM_DB_RED`, `EPWM_DB_FED`).
        * `enableDelayMode`: `true` para aplicar o atraso, `false` para ignorar o atraso.
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o bit `OUT_MODE` no registrador `DBCTL` para habilitar ou desabilitar o atraso Dead Band.

* **`EPWM_setDeadBandDelayPolarity(uint32_t base, EPWM_DeadBandDelayMode delayMode, EPWM_DeadBandPolarity polarity)`**
    * **Propósito:** Define a polaridade do atraso Dead Band.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `delayMode`: Tipo de atraso Dead Band.
        * `polarity`: Polaridade do sinal atrasado (e.g., `EPWM_DB_POLARITY_ACTIVE_HIGH`, `EPWM_DB_POLARITY_ACTIVE_LOW`).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o bit `POLSEL` no registrador `DBCTL` para definir a polaridade do sinal atrasado.

* **`EPWM_setRisingEdgeDeadBandDelayInput(uint32_t base, uint16_t input)`**
    * **Propósito:** Define o sinal de entrada para o atraso Dead Band de borda de subida.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `input`: Sinal de entrada (e.g., `EPWM_DB_INPUT_EPWMA`, `EPWM_DB_INPUT_EPWMB`).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o bit `IN_MODE` no registrador `DBCTL` para selecionar a entrada para o atraso de borda de subida.

* **`EPWM_setFallingEdgeDeadBandDelayInput(uint32_t base, uint16_t input)`**
    * **Propósito:** Define o sinal de entrada para o atraso Dead Band de borda de descida.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `input`: Sinal de entrada (e.g., `EPWM_DB_INPUT_EPWMA`, `EPWM_DB_INPUT_EPWMB`, `EPWM_DB_INPUT_DB_RED`).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura os bits `IN_MODE` e `DEDB_MODE` no registrador `DBCTL` para selecionar a entrada para o atraso de borda de descida.

* **`EPWM_setDeadBandControlShadowLoadMode(uint32_t base, EPWM_DeadBandControlLoadMode loadMode)`**
    * **Propósito:** Define o modo de carregamento sombra do registrador de controle Dead Band.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `loadMode`: Modo de carregamento sombra (e.g., `EPWM_DB_LOAD_ON_CNTR_ZERO`).
    * **Retorno:** Nenhum.
    * **Descrição:** Habilita e configura o modo de carregamento sombra para o registrador de controle Dead Band (`DBCTL`).

* **`EPWM_disableDeadBandControlShadowLoadMode(uint32_t base)`**
    * **Propósito:** Desabilita o modo de carregamento sombra do registrador de controle Dead Band.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Desabilita o modo de carregamento sombra para o registrador de controle Dead Band.

* **`EPWM_setRisingEdgeDelayCountShadowLoadMode(uint32_t base, EPWM_RisingEdgeDelayLoadMode loadMode)`**
    * **Propósito:** Define o modo de carregamento sombra do atraso de borda de subida (RED).
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `loadMode`: Modo de carregamento sombra (e.g., `EPWM_RED_LOAD_ON_CNTR_ZERO`).
    * **Retorno:** Nenhum.
    * **Descrição:** Habilita e configura o modo de carregamento sombra para o registrador de atraso de borda de subida (`DBRED`).

* **`EPWM_disableRisingEdgeDelayCountShadowLoadMode(uint32_t base)`**
    * **Propósito:** Desabilita o modo de carregamento sombra do atraso de borda de subida (RED).
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Desabilita o modo de carregamento sombra para o registrador de atraso de borda de subida.

* **`EPWM_setFallingEdgeDelayCountShadowLoadMode(uint32_t base, EPWM_FallingEdgeDelayLoadMode loadMode)`**
    * **Propósito:** Define o modo de carregamento sombra do atraso de borda de descida (FED).
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `loadMode`: Modo de carregamento sombra (e.g., `EPWM_FED_LOAD_ON_CNTR_ZERO`).
    * **Retorno:** Nenhum.
    * **Descrição:** Habilita e configura o modo de carregamento sombra para o registrador de atraso de borda de descida (`DBFED`).

* **`EPWM_disableFallingEdgeDelayCountShadowLoadMode(uint32_t base)`**
    * **Propósito:** Desabilita o modo de carregamento sombra do atraso de borda de descida (FED).
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Desabilita o modo de carregamento sombra para o registrador de atraso de borda de descida.

* **`EPWM_setDeadBandCounterClock(uint32_t base, EPWM_DeadBandClockMode clockMode)`**
    * **Propósito:** Define a taxa de clock do contador Dead Band.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `clockMode`: Modo de clock do contador Dead Band (e.g., `EPWM_DB_COUNTER_CLOCK_FULL_CYCLE`, `EPWM_DB_COUNTER_CLOCK_HALF_CYCLE`).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura o registrador `DBCTL` para definir a taxa de clock do contador Dead Band em relação ao `TBCLK`.

* **`EPWM_setRisingEdgeDelayCount(uint32_t base, uint16_t redCount)`**
    * **Propósito:** Define a contagem de atraso de borda de subida (RED).
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `redCount`: Contagem de atraso RED (menor que `0x4000U`).
    * **Retorno:** Nenhum.
    * **Descrição:** Define o valor da contagem RED no registrador `DBRED`.

* **`EPWM_setFallingEdgeDelayCount(uint32_t base, uint16_t fedCount)`**
    * **Propósito:** Define a contagem de atraso de borda de descida (FED).
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `fedCount`: Contagem de atraso FED (menor que `0x4000U`).
    * **Retorno:** Nenhum.
    * **Descrição:** Define o valor da contagem FED no registrador `DBFED`.

---

### 4.6 Submódulo Minimum Dead-Band and Illegal Combination

**Propósito:** O submódulo EPWM Minimum Dead-Band é semelhante ao submódulo Dead-Band na aplicação de um atraso configurável entre os módulos PWM. O objetivo deste recurso é garantir que os gates não estejam ligados ao mesmo tempo e prevenir um curto-circuito.

**Funcionamento:** No diagrama abaixo, quando uma borda de descida ocorre em `EPWMxA`, o atraso mínimo definido será aplicado à saída de `EPWMxB`. Quando uma borda de descida ocorre em `EPWMxB`, um atraso mínimo é aplicado a `EPWMxA`.

**Forma de Onda de Tempo Morto Mínimo (Conceitual):**

[A imagem ilustra a aplicação de tempo morto mínimo entre sinais complementares.]

À medida que o PWM aumenta em configurabilidade, há um módulo de lógica de combinação ilegal (ICL) que receberá um conjunto de combinações indesejadas e produzirá um determinado estado. No diagrama abaixo, a lógica de combinação ilegal é uma tabela verdade que produzirá `EPWMA`/`EPWMB` alto ou baixo com base no estado das entradas.

**Diagrama de Bloco ICL (Conceitual):**

[A imagem ilustra o funcionamento da lógica de combinação ilegal (ICL).]

Um exemplo de cenário de combinação indesejada para `OUT` poderia ser `EPWMxA_MINDB` alto, `EPWMxB_MINDB` baixo, e a saída do ICL X-BAR alta. O valor de `OUT` poderia ser codificado para ser baixo. Para uma aplicação, isso garante que certas combinações serão direcionadas para um estado conhecido.

**EPWM MINDB and ICL X-BARs:**

* **Propósito:** Os X-BARs de Tempo Morto Mínimo (MINDB) e Lógica de Combinação Ilegal (ICL) são novos no F28P65x. Seu propósito é expandir a funcionalidade dos submódulos MINDB e ICL do EPWM roteando sinais do módulo de lógica de emulação de diodo, módulo de tempo morto mínimo e CLB.
* **Funcionalidade:** Sua funcionalidade é semelhante ao módulo Input X-BAR, o que significa que apenas uma única entrada para os X-BARs pode ser roteada para a saída, ao contrário do Output X-BAR, por exemplo, onde múltiplas entradas podem ser combinadas como um OR lógico e enviadas para a saída.

**Diagrama de Bloco MINDB e ICL X-BAR (Conceitual):**

[A imagem ilustra a arquitetura dos X-BARs MINDB e ICL.]

---

### 4.7 Submódulo Valley Switching

**Propósito:** Este submódulo é usado para detecção de vale e comutação de vale, que são técnicas importantes em certas aplicações de conversão de potência (como PFC em modo de condução crítica) para reduzir perdas de comutação.

**Visão Geral:** O Valley Switching permite que o EPWM capture o valor do contador da base de tempo em um ponto específico (o "vale") de uma forma de onda e, opcionalmente, adicione um atraso a esse ponto para sincronizar a comutação.

**Funções da Driverlib:**

* **`EPWM_enableValleyCapture(uint32_t base)`**
    * **Propósito:** Habilita o modo de captura de vale.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Define o bit `VCAPE` no registrador `VCAPCTL` para habilitar a captura de vale.

* **`EPWM_disableValleyCapture(uint32_t base)`**
    * **Propósito:** Desabilita o modo de captura de vale.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa o bit `VCAPE` no registrador `VCAPCTL`.

* **`EPWM_startValleyCapture(uint32_t base)`**
    * **Propósito:** Inicia a sequência de captura de vale.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Define o bit `VCAPSTART` no registrador `VCAPCTL` para iniciar a sequência de captura de vale. Deve ser usado após configurar a fonte de disparo para software.

* **`EPWM_setValleyTriggerSource(uint32_t base, EPWM_ValleyTriggerSource trigger)`**
    * **Propósito:** Define o valor do disparo que inicia a sequência de captura de vale.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `trigger`: Disparo do contador de vale (e.g., `EPWM_VALLEY_TRIGGER_EVENT_SOFTWARE`, `EPWM_VALLEY_TRIGGER_EVENT_CNTR_ZERO`).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura os bits `TRIGSEL` no registrador `VCAPCTL` para selecionar a fonte do disparo para a captura de vale.

* **`EPWM_setValleyTriggerEdgeCounts(uint32_t base, uint16_t startCount, uint16_t stopCount)`**
    * **Propósito:** Define o número de eventos de disparo necessários para iniciar e parar a contagem de captura de vale.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `startCount`: Contagem para iniciar a captura (0-15).
        * `stopCount`: Contagem para parar a captura (0-15).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura os bits `STARTEDGE` e `STOPEDGE` no registrador `VCNTCFG`.

* **`EPWM_enableValleyHWDelay(uint32_t base)`**
    * **Propósito:** Habilita o atraso de comutação de vale por hardware.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Define o bit `EDGEFILTDLYSEL` no registrador `VCAPCTL` para habilitar o atraso de comutação de vale.

* **`EPWM_disableValleyHWDelay(uint32_t base)`**
    * **Propósito:** Desabilita o atraso de comutação de vale por hardware.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa o bit `EDGEFILTDLYSEL` no registrador `VCAPCTL`.

* **`EPWM_setValleySWDelayValue(uint32_t base, uint16_t delayOffsetValue)`**
    * **Propósito:** Define o valor de atraso de vale definido por software.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `delayOffsetValue`: Valor de offset de atraso definido por software.
    * **Retorno:** Nenhum.
    * **Descrição:** Escreve o valor no registrador `SWVDELVAL`.

* **`EPWM_setValleyDelayDivider(uint32_t base, EPWM_ValleyDelayMode delayMode)`**
    * **Propósito:** Define o modo de atraso de vale.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `delayMode`: Modo de atraso de vale (e.g., `EPWM_VALLEY_DELAY_MODE_SW_DELAY`).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura os bits `VDELAYDIV` no registrador `VCAPCTL`.

* **`EPWM_getValleyEdgeStatus(uint32_t base, EPWM_ValleyCounterEdge edge)`**
    * **Propósito:** Retorna o status da borda de início ou parada do vale.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `edge`: Borda de início ou parada (e.g., `EPWM_VALLEY_COUNT_START_EDGE`).
    * **Retorno:** `true` se a borda especificada ocorreu, `false` caso contrário.
    * **Descrição:** Retorna o status dos bits `STARTEDGESTS` ou `STOPEDGESTS` no registrador `VCNTCFG`.

* **`EPWM_getValleyCount(uint32_t base)`**
    * **Propósito:** Retorna o valor do contador de vale.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** O valor da contagem da base de tempo do vale.
    * **Descrição:** Retorna o valor do registrador `VCNTVAL`.

* **`EPWM_getValleyHWDelay(uint32_t base)`**
    * **Propósito:** Retorna o valor de atraso de vale por hardware.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** A contagem de atraso de vale.
    * **Descrição:** Retorna o valor do registrador `HWVDELVAL`.

---

### 4.8 Submódulo Global Load

**Propósito:** O submódulo Global Load permite que múltiplos registradores de um ou mais módulos EPWM sejam atualizados de forma síncrona em um evento de carregamento global. Isso é útil para garantir que as configurações do PWM sejam aplicadas de forma coordenada.

**Visão Geral:** Em vez de atualizar cada registrador individualmente, o Global Load permite que você prepare os novos valores em registradores sombra e, em seguida, os carregue para os registradores ativos de forma atômica em um evento de disparo.

**Funções da Driverlib:**

* **`EPWM_enableGlobalLoad(uint32_t base)`**
    * **Propósito:** Habilita o modo de carregamento sombra global para registradores.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Define o bit `GLD` no registrador `GLDCTL` para controlar o carregamento sombra globalmente.

* **`EPWM_disableGlobalLoad(uint32_t base)`**
    * **Propósito:** Desabilita o modo de carregamento sombra global para registradores.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa o bit `GLD` no registrador `GLDCTL`.

* **`EPWM_setGlobalLoadTrigger(uint32_t base, EPWM_GlobalLoadTrigger loadTrigger)`**
    * **Propósito:** Define o pulso que causa o carregamento sombra global.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `loadTrigger`: Pulso que causa o carregamento global (e.g., `EPWM_GL_LOAD_PULSE_CNTR_ZERO`, `EPWM_GL_LOAD_PULSE_SYNC`).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura os bits `GLDMODE` no registrador `GLDCTL` para selecionar o evento que aciona o carregamento global.

* **`EPWM_setGlobalLoadEventPrescale(uint32_t base, uint16_t prescalePulseCount)`**
    * **Propósito:** Define o número de contagens de eventos de pulso de carregamento global.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `prescalePulseCount`: Contagem de eventos de pulso (0-7).
    * **Retorno:** Nenhum.
    * **Descrição:** Configura os bits `GLDPRD` no registrador `GLDCTL` para definir quantos eventos de pulso de carregamento global devem ocorrer antes que um pulso de carregamento seja emitido.

* **`EPWM_getGlobalLoadEventCount(uint32_t base)`**
    * **Propósito:** Retorna o número de contagens de eventos de pulso de carregamento global.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** O número de eventos de pulso de carregamento global que ocorreram.
    * **Descrição:** Retorna o valor dos bits `GLDCNT` no registrador `GLDCTL`.

* **`EPWM_disableGlobalLoadOneShotMode(uint32_t base)`**
    * **Propósito:** Habilita o carregamento global contínuo de sombra para ativo.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa o bit `OSHTMODE` no registrador `GLDCTL` para permitir carregamento contínuo.

* **`EPWM_enableGlobalLoadOneShotMode(uint32_t base)`**
    * **Propósito:** Habilita o carregamento global de sombra para ativo em modo one-shot.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Define o bit `OSHTMODE` no registrador `GLDCTL` para habilitar o carregamento one-shot.

* **`EPWM_setGlobalLoadOneShotLatch(uint32_t base)`**
    * **Propósito:** Define um latch de carregamento global one-shot.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Define o bit `OSHTLD` no registrador `GLDCTL2` para propagar um pulso de carregamento one-shot.

* **`EPWM_forceGlobalLoadOneShotEvent(uint32_t base)`**
    * **Propósito:** Força um evento de carregamento global one-shot por software.
    * **Parâmetros:** `base` (endereço base do módulo EPWM).
    * **Retorno:** Nenhum.
    * **Descrição:** Define o bit `GFRCLD` no registrador `GLDCTL2` para forçar um carregamento one-shot.

* **`EPWM_enableGlobalLoadRegisters(uint32_t base, uint16_t loadRegister)`**
    * **Propósito:** Habilita um registrador para ser carregado globalmente.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `loadRegister`: Registrador a ser carregado globalmente (e.g., `EPWM_GL_REGISTER_TBPRD_TBPRDHR`, `EPWM_GL_REGISTER_CMPA_CMPAHR`).
    * **Retorno:** Nenhum.
    * **Descrição:** Define os bits correspondentes no registrador `GLDCFG` para incluir o registrador no carregamento global.

* **`EPWM_disableGlobalLoadRegisters(uint32_t base, uint16_t loadRegister)`**
    * **Propósito:** Desabilita um registrador de ser carregado globalmente.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `loadRegister`: Registrador a ser desabilitado do carregamento global.
    * **Retorno:** Nenhum.
    * **Descrição:** Limpa os bits correspondentes no registrador `GLDCFG` para excluir o registrador do carregamento global.

---

### 4.9 Submódulo Lock Registers

**Propósito:** O submódulo Lock Registers permite proteger grupos de registradores específicos do EPWM contra modificações acidentais, exigindo uma sequência de desbloqueio para acesso. Isso é crucial para a segurança e estabilidade do sistema em aplicações críticas.

**Visão Geral:** Certos grupos de registradores do EPWM são protegidos por `EALLOW` (Enable Access to EALLOW-protected registers). A função de bloqueio adiciona uma camada extra de proteção, impedindo modificações mesmo com `EALLOW` habilitado, a menos que uma chave de desbloqueio específica seja escrita.

**Funções da Driverlib:**

* **`EPWM_lockRegisters(uint32_t base, EPWM_LockRegisterGroup registerGroup)`**
    * **Propósito:** Bloqueia grupos de registradores protegidos por `EALLOW`.
    * **Parâmetros:**
        * `base`: Endereço base do módulo EPWM.
        * `registerGroup`: Grupo de registradores a ser bloqueado (e.g., `EPWM_REGISTER_GROUP_GLOBAL_LOAD`, `EPWM_REGISTER_GROUP_TRIP_ZONE`).
    * **Retorno:** Nenhum.
    * **Descrição:** Escreve uma chave específica no registrador `LOCK` para bloquear o grupo de registradores especificado. Uma vez bloqueados, esses registradores não podem ser modificados até que sejam explicitamente desbloqueados.

---

## Seção 5: High Resolution PWM (HRPWM)

**Propósito:** O módulo EPWM é capaz de aumentar significativamente suas capacidades de resolução de tempo em relação ao PWM digital convencional. Isso é conseguido adicionando extensões de 8 bits aos registradores de comparação de contador (`CMPxHR`), registrador de período da base de tempo (`TBPRDHR`) e registrador de fase da base de tempo (`TBPHSHR`) para fornecer uma granularidade de tempo mais fina para o controle de posicionamento de borda. Isso é conhecido como PWM de alta resolução (HRPWM) e é baseado na tecnologia de posicionador de microborda (MEP).

**Tecnologia MEP:** A lógica MEP é capaz de posicionar uma borda muito finamente, subdividindo um clock de sistema grosso do gerador PWM convencional com precisão de passo de tempo da ordem de 150 picosegundos (consulte a documentação do seu dispositivo para especificações). Um diagnóstico de software de autoavaliação (SFO Library) é usado para determinar se a lógica MEP está funcionando otimamente, e pode calibrar a lógica MEP em todas as condições de operação para contabilizar variações causadas por temperatura, tensão e processo. O HRPWM é tipicamente usado quando a resolução PWM cai abaixo de aproximadamente 9 ou 10 bits.

**Resumo do HRPWM:**

* Aumenta significativamente a resolução do PWM digital convencionalmente derivado.
* Adiciona extensões de 8 bits aos registradores de comparação de contador (`CMPxHR`), registrador de período da base de tempo (`TBPRDHR`) e registrador de fase da base de tempo (`TBPHSHR`) para controle de posicionamento de microborda (MEP).
* Tipicamente usado quando a resolução PWM cai abaixo de ~9-10 bits.

**Observação:** Nem todas as saídas EPWM suportam o recurso HRPWM. Consulte a folha de dados do seu dispositivo para obter detalhes.

---

## Seção 6: Recursos

### 6.1 Materiais Relacionados ao EPWM

**Fundamentais**

* Real-Time Control Reference Guide
* Primeiros Passos
* C2000 ePWM Developer’s Guide Application Report - Aplicável apenas a: F280013x, F280015x, F28002x, F28003x, F28004x, F2807x, F2837x, F2838x, F28P55x, F28P65x
* Enhanced Pulse Width Modulator (ePWM) Training for C2000 MCUs - Aplicável apenas a: F280013x, F280015x, F28002x, F28003x, F28004x, F2837x, F2838x, F28P55x, F28P65x
* Getting Started with the C2000 ePWM Module
* C28x Academy - EPWM
* Using the Enhanced Pulse Width Modulator (ePWM) Module Application Report

**Expert**

* C2000 real-time microcontrollers - Reference designs
* CRM/ZVS PFC Implementation Based on C2000 Type-4 PWM Module Application Report - Aplicável apenas a: F280013x, F280015x, F28002x, F28003x, F28004x, F2837x, F2838x, F28P55x
* Leverage New Type ePWM Features for Multiple Phase Control Application Report - Aplicável apenas a: F280013x, F280015x, F28002x, F28003x, F28004x, F2837x, F2838x, F28P55x, F28P65x
```
