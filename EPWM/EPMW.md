# EPWM - Modulação por Largura de Pulso Aprimorada

## Introdução

Este capítulo aprofunda-se nas complexidades do driver do módulo Enhanced Pulse Width Modulation (EPWM) para o microcontrolador Texas Instruments DSP28004x[cite: 1]. O EPWM é um periférico poderoso capaz de gerar sinais de modulação por largura de pulso precisos, essenciais para aplicações como controle de motores, eletrônica de potência e conversão digital-analógica[cite: 2]. Este capítulo cobrirá as funções fundamentais fornecidas no driver, permitindo que os desenvolvedores configurem e utilizem efetivamente o módulo EPWM[cite: 3].

## Seção 1: Funções do Driver EPWM

Esta seção detalha as funções disponíveis no driver EPWM, fornecendo explicações sobre seu propósito, parâmetros e uso[cite: 4].

### 1.1 `EPWM_setEmulationMode()`

* **Propósito**: Configura o comportamento do módulo EPWM durante o modo de emulação (por exemplo, quando o depurador pausa a CPU)[cite: 5].
* **Sintaxe**[cite: 6]:
    ```c
    void EPWM_setEmulationMode(uint32_t base, EPWM_EmulationMode emulationMode);
    ```
* **Parâmetros**:
    * `base`: O endereço base do módulo EPWM[cite: 6]. Use as macros predefinidas (por exemplo, `EPWM1_BASE`, `EPWM2_BASE`) para instâncias EPWM específicas[cite: 7].
    * `emulationMode`: Especifica o modo de emulação[cite: 7]. Deve ser um dos valores do enum `EPWM_EmulationMode`[cite: 8]:
        * `EPWM_EMULATION_FREE_RUN`: O contador EPWM continua a funcionar no modo de emulação[cite: 8].
        * `EPWM_EMULATION_STOP_AFTER_NEXT`: O contador EPWM para após o próximo ciclo no modo de emulação[cite: 9].
        * `EPWM_EMULATION_STOP_IMMEDIATELY`: O contador EPWM para imediatamente no modo de emulação[cite: 10].
* **Valor de Retorno**: Nenhum (função void)[cite: 10].
* **Descrição**: Esta função define os bits `FREE_SOFT` no registrador `TBCTL`, controlando como o temporizador EPWM se comporta quando a CPU é pausada pelo depurador[cite: 11, 12]. Isso é crucial para depurar sistemas em tempo real[cite: 13].
* **Exemplo**[cite: 13]:
    ```c
    EPWM_setEmulationMode(EPWM1_BASE, EPWM_EMULATION_STOP_AFTER_NEXT);
    ```
    Este exemplo configura o EPWM1 para parar seu contador após o ciclo atual quando o depurador pausa a CPU[cite: 14].

### 1.2 `EPWM_configureSignal()`

* **Propósito**: Configura os parâmetros operacionais primários de um sinal EPWM, incluindo frequência, ciclo de trabalho e modo de contador[cite: 15].
* **Sintaxe**[cite: 16]:
    ```c
    void EPWM_configureSignal(uint32_t base, const EPWM_SignalParams *signalParams);
    ```
* **Parâmetros**:
    * `base`: O endereço base do módulo EPWM (por exemplo, `EPWM1_BASE`)[cite: 16].
    * `signalParams`: Um ponteiro para uma estrutura `EPWM_SignalParams` constante que contém as configurações para o sinal EPWM[cite: 17].
* **Valor de Retorno**: Nenhum (função void)[cite: 18].
* **Descrição**: Esta é a função de configuração EPWM mais abrangente[cite: 18]. Ela calcula e define os valores de registro necessários com base nos parâmetros fornecidos[cite: 19]. Ela lida com a pré-escalagem do clock da base de tempo, modo de contador, período, valores de comparação e qualificadores de ação para gerar as formas de onda PWM desejadas em `EPWMxA` e `EPWMxB`[cite: 20].
* **A Estrutura `EPWM_SignalParams`**[cite: 21]:
    ```c
    typedef struct
    {
        float32_t sysClkInHz; // Frequência do clock do sistema em Hz [cite: 21]
        float32_t freqInHz; // Frequência PWM desejada em Hz [cite: 22]
        EPWM_TimeBaseClockDivider tbClkDiv; // Divisor do clock da base de tempo [cite: 22]
        EPWM_TimeBaseHighSpeedClockDivider tbHSClkDiv; // Divisor do clock de alta velocidade da base de tempo [cite: 23]
        EPWM_CounterMode tbCtrMode; // Modo de contador da base de tempo [cite: 24]
        float32_t dutyValA; // Valor do ciclo de trabalho para EPWMxA (0.0 a 1.0) [cite: 25]
        float32_t dutyValB; // Valor do ciclo de trabalho para EPWMxB (0.0 a 1.0) [cite: 26]
        bool invertSignalB; // Invertir a saída EPWMxB (true ou false) [cite: 27]
    } EPWM_SignalParams;
    ```
* **Principais Etapas de Configuração Executadas por `EPWM_configureSignal()`**:
    1.  **Configuração do Clock da Base de Tempo**:
        * Chama `EPWM_setClockPrescaler()` para definir a divisão do clock para o contador da base de tempo[cite: 28].
    2.  **Modo de Contador da Base de Tempo**:
        * Chama `EPWM_setTimeBaseCounterMode()` para definir o modo de contador (crescente, decrescente ou crescente-decrescente)[cite: 29].
    3.  **Cálculo de `TBPRD`, `CMPA`, `CMPB`**:
        * Calcula o período da base de tempo (`TBPRD`) e os valores de comparação (`CMPA`, `CMPB`) com base no clock do sistema, frequência PWM desejada, ciclos de trabalho e modo de contador[cite: 30]. Os cálculos garantem que a frequência e o ciclo de trabalho corretos sejam alcançados[cite: 31].
    4.  **Configuração Inicial**:
        * Desabilita o carregamento de deslocamento de fase, define o deslocamento de fase para 0 e inicializa o contador da base de tempo para 0[cite: 32].
    5.  **Carregamento do Registro Sombra**:
        * Configura o carregamento do registro sombra para `CMPA` e `CMPB` para ocorrer no evento zero do contador da base de tempo[cite: 32]. Isso garante atualizações sincronizadas dos valores de comparação[cite: 33].
    6.  **Definição do Valor de Comparação**:
        * Chama `EPWM_setCounterCompareValue()` para definir os valores calculados de `CMPA` e `CMPB`[cite: 33].
    7.  **Configuração do Qualificador de Ação**:
        * Usa `EPWM_setActionQualifierAction()` para definir as ações tomadas nas saídas EPWM (`EPWMxA` e `EPWMxB`) com base nos eventos do contador da base de tempo (zero, comparação A, comparação B)[cite: 34]. É assim que a forma de onda PWM é gerada[cite: 35]. A lógica muda com base no modo de contador selecionado e se o `EPWMxB` precisa ser invertido[cite: 36].
* **Exemplo**[cite: 36]:
    ```c
    EPWM_SignalParams myEPWMConfig;
    myEPWMConfig.sysClkInHz = 200e6; // Clock do sistema de 200 MHz [cite: 37]
    myEPWMConfig.freqInHz = 10e3; // Frequência PWM de 10 kHz [cite: 37]
    myEPWMConfig.tbClkDiv = EPWM_CLOCK_DIVIDER_1; [cite: 37]
    myEPWMConfig.tbHSClkDiv = EPWM_HSCLOCK_DIVIDER_1; [cite: 38]
    myEPWMConfig.tbCtrMode = EPWM_COUNTER_MODE_UP_DOWN; [cite: 38]
    myEPWMConfig.dutyValA = 0.25f; // Ciclo de trabalho de 25% para EPWMxA [cite: 38]
    myEPWMConfig.dutyValB = 0.75f; // Ciclo de trabalho de 75% para EPWMxB [cite: 39]
    myEPWMConfig.invertSignalB = false; [cite: 39]

    EPWM_configureSignal(EPWM2_BASE, &myEPWMConfig);
    ```
    Este exemplo configura o EPWM2 para gerar um sinal PWM de 10 kHz com ciclo de trabalho de 25% em `EPWMxA` e 75% em `EPWMxB`, usando o modo de contagem crescente-decrescente e um clock de sistema de 200 MHz[cite: 40].

## Seção 2: Funções Auxiliares (Mencionadas dentro de `EPWM_configureSignal`)

Estas funções são chamadas internamente por `EPWM_configureSignal()` para lidar com tarefas de configuração específicas[cite: 41]:

### 2.1 `EPWM_setClockPrescaler()`

* **Propósito**: Configura os divisores de clock para o contador da base de tempo do módulo EPWM[cite: 41]. Isso permite controlar a frequência do clock de contagem do temporizador, que por sua vez afeta o período máximo e a resolução do PWM[cite: 42].
* **Sintaxe**[cite: 43]:
    ```c
    void EPWM_setClockPrescaler(uint32_t base,
                                EPWM_TimeBaseClockDivider tbClkDiv,
                                EPWM_TimeBaseHighSpeedClockDivider tbHSClkDiv);
    ```
* **Parâmetros**:
    * `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`)[cite: 43].
    * `tbClkDiv`: O divisor do clock da base de tempo principal[cite: 44]. Deve ser um dos valores do enum `EPWM_TimeBaseClockDivider` (e.g., `EPWM_CLOCK_DIVIDER_1`, `EPWM_CLOCK_DIVIDER_2`, etc.)[cite: 45].
    * `tbHSClkDiv`: O divisor do clock de alta velocidade da base de tempo[cite: 46]. Deve ser um dos valores do enum `EPWM_TimeBaseHighSpeedClockDivider` (e.g., `EPWM_HSCLOCK_DIVIDER_1`, `EPWM_HSCLOCK_DIVIDER_2`, etc.)[cite: 47].
* **Valor de Retorno**: Nenhum (função void)[cite: 47].
* **Descrição**: Esta função configura os bits `CLKDIV` e `HSPCLKDIV` no registrador `TBCTL` do EPWM[cite: 48]. A frequência final do clock da base de tempo é determinada pela divisão do clock do sistema por esses dois prescalers[cite: 49]. Uma frequência de clock da base de tempo mais baixa permite períodos de PWM mais longos, enquanto uma frequência mais alta oferece maior resolução[cite: 50].
* **Exemplo**[cite: 51]:
    ```c
    // Configura o EPWM1 para dividir o clock do sistema por 2 (TBCLK)
    // e o TBCLK por 4 (HSPCLK)
    EPWM_setClockPrescaler(EPWM1_BASE, EPWM_CLOCK_DIVIDER_2, EPWM_HSCLOCK_DIVIDER_4);
    ```

### 2.2 `EPWM_setTimeBaseCounterMode()`

* **Propósito**: Define o modo de contagem do contador da base de tempo do módulo EPWM[cite: 52]. O modo de contagem determina como o contador do temporizador opera (crescente, decrescente ou crescente-decrescente), o que é fundamental para a geração de diferentes tipos de formas de onda PWM[cite: 53].
* **Sintaxe**[cite: 54]:
    ```c
    void EPWM_setTimeBaseCounterMode(uint32_t base, EPWM_CounterMode counterMode);
    ```
* **Parâmetros**:
    * `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`)[cite: 54].
    * `counterMode`: O modo de contagem desejado[cite: 55]. Deve ser um dos valores do enum `EPWM_CounterMode`[cite: 55]:
        * `EPWM_COUNTER_MODE_UP`: O contador conta de 0 até o período (`TBPRD`)[cite: 55].
        * `EPWM_COUNTER_MODE_DOWN`: O contador conta do período (`TBPRD`) até 0[cite: 57].
        * `EPWM_COUNTER_MODE_UP_DOWN`: O contador conta de 0 até o período (`TBPRD`) e depois de volta para 0[cite: 57].
        * `EPWM_COUNTER_MODE_STOP_FREEZE`: O contador para e congela seu valor atual[cite: 57].
* **Valor de Retorno**: Nenhum (função void)[cite: 58].
* **Descrição**: Esta função configura os bits `CTRMODE` no registrador `TBCTL` do EPWM[cite: 58]. A escolha do modo de contagem impacta diretamente como os eventos de comparação (`CMPA`, `CMPB`) são usados para alternar os estados das saídas PWM (`EPWMxA`, `EPWMxB`), permitindo a criação de formas de onda de borda única, borda dupla ou simétricas[cite: 59].
* **Exemplo**[cite: 60]:
    ```c
    // Configura o EPWM1 para operar no modo de contagem crescente-decrescente
    EPWM_setTimeBaseCounterMode(EPWM1_BASE, EPWM_COUNTER_MODE_UP_DOWN);
    ```

### 2.3 `EPWM_setTimeBasePeriod()`

* **Propósito**: Define o valor do período para o contador da base de tempo do módulo EPWM[cite: 61]. Este valor determina o limite superior (ou limite superior e inferior, dependendo do modo de contagem) para o contador, estabelecendo assim a frequência fundamental do sinal PWM[cite: 62].
* **Sintaxe**[cite: 63]:
    ```c
    void EPWM_setTimeBasePeriod(uint32_t base, uint16_t period);
    ```
* **Parâmetros**:
    * `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`)[cite: 63].
    * `period`: O valor do período para o contador da base de tempo[cite: 64]. Este valor é carregado no registrador `TBPRD`[cite: 64]. O valor máximo é 0xFFFF (65535)[cite: 65].
* **Valor de Retorno**: Nenhum (função void)[cite: 65].
* **Descrição**: Esta função escreve o valor fornecido no registrador `TBPRD` (Time Base Period) do EPWM[cite: 66]. O contador da base de tempo (`TBCTR`) conta de 0 até este valor (no modo crescente), ou de `period` até 0 (no modo decrescente), ou de 0 até `period` e de volta para 0 (no modo crescente-decrescente)[cite: 67]. A frequência do PWM é inversamente proporcional ao valor do período e à frequência do clock da base de tempo[cite: 68, 69].
* **Exemplo**[cite: 70]:
    ```c
    // Define o período do EPWM1 para 10000
    EPWM_setTimeBasePeriod(EPWM1_BASE, 10000);
    ```

### 2.4 `EPWM_disablePhaseShiftLoad()`

* **Propósito**: Desabilita o carregamento do valor de deslocamento de fase para o contador da base de tempo do módulo EPWM[cite: 71]. Quando desabilitado, o contador da base de tempo não é inicializado com um valor de deslocamento de fase ao iniciar ou reiniciar[cite: 72].
* **Sintaxe**[cite: 73]:
    ```c
    void EPWM_disablePhaseShiftLoad(uint32_t base);
    ```
* **Parâmetros**:
    * `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`)[cite: 73].
* **Valor de Retorno**: Nenhum (função void)[cite: 74].
* **Descrição**: Esta função limpa o bit `PHSEN` no registrador `TBCTL` do EPWM[cite: 74]. Ao desabilitar o carregamento de deslocamento de fase, o contador da base de tempo (`TBCTR`) sempre iniciará sua contagem a partir de 0 (ou do valor de reset padrão) em vez de um valor de fase específico[cite: 75]. Isso é útil para garantir que vários módulos EPWM iniciem suas contagens de forma síncrona ou para configurações onde o deslocamento de fase não é necessário[cite: 76].
* **Exemplo**[cite: 77]:
    ```c
    // Desabilita o carregamento de deslocamento de fase para o EPWM1
    EPWM_disablePhaseShiftLoad(EPWM1_BASE);
    ```

### 2.5 `EPWM_setPhaseShift()`

* **Propósito**: Define o valor de deslocamento de fase para o contador da base de tempo do módulo EPWM[cite: 78]. Este valor é usado para inicializar o contador da base de tempo quando o carregamento de fase está habilitado, permitindo a sincronização de múltiplos módulos EPWM com um atraso específico[cite: 79, 80].
* **Sintaxe**[cite: 81]:
    ```c
    void EPWM_setPhaseShift(uint32_t base, uint16_t phaseShift);
    ```
* **Parâmetros**:
    * `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`)[cite: 81].
    * `phaseShift`: O valor de deslocamento de fase[cite: 82]. Este valor é carregado no registrador `TBPHS`[cite: 82]. O valor máximo é 0xFFFF (65535)[cite: 82].
* **Valor de Retorno**: Nenhum (função void)[cite: 83].
* **Descrição**: Esta função escreve o valor fornecido no registrador `TBPHS` (Time Base Phase) do EPWM[cite: 83]. Quando o carregamento de fase está habilitado (o bit `PHSEN` no `TBCTL` está definido), o contador da base de tempo (`TBCTR`) será inicializado com este valor no evento de sincronização (por exemplo, um evento de sincronização de software ou um evento de sincronização de outro EPWM)[cite: 84]. Isso é fundamental para controlar a relação de fase entre diferentes saídas PWM[cite: 85].
* **Exemplo**[cite: 86]:
    ```c
    // Define um deslocamento de fase de 500 para o EPWM1
    EPWM_setPhaseShift(EPWM1_BASE, 500U);
    ```

### 2.6 `EPWM_setTimeBaseCounter()`

* **Propósito**: Define o valor atual do contador da base de tempo do módulo EPWM[cite: 87]. Isso permite inicializar o contador para um valor específico ou modificá-lo em tempo de execução[cite: 88].
* **Sintaxe**[cite: 88]:
    ```c
    void EPWM_setTimeBaseCounter(uint32_t base, uint16_t counterValue);
    ```
* **Parâmetros**[cite: 89]:
    * `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`)[cite: 89].
    * `counterValue`: O valor para o qual o contador da base de tempo (`TBCTR`) será definido[cite: 90]. O valor máximo é 0xFFFF (65535)[cite: 91].
* **Valor de Retorno**: Nenhum (função void)[cite: 91].
* **Descrição**: Esta função escreve o `counterValue` fornecido diretamente no registrador `TBCTR` (Time Base Counter) do EPWM[cite: 92, 93]. Definir o contador da base de tempo pode ser útil para[cite: 94]:
    * Inicializar o contador em um ponto específico de um ciclo PWM[cite: 95].
    * Sincronizar o contador com outros eventos ou módulos de hardware[cite: 95].
    * Implementar técnicas avançadas de controle de PWM que exigem manipulação direta do contador[cite: 96].
* **Exemplo**[cite: 97]:
    ```c
    // Define o contador da base de tempo do EPWM1 para 0
    EPWM_setTimeBaseCounter(EPWM1_BASE, 0U); [cite: 97]
    // Define o contador da base de tempo do EPWM2 para 250
    EPWM_setTimeBaseCounter(EPWM2_BASE, 250U); [cite: 98]
    ```

### 2.7 `EPWM_setCounterCompareShadowLoadMode()`

* **Propósito**: Configura o modo de carregamento sombra para os registradores de comparação (`CMPA` e `CMPB`) do módulo EPWM[cite: 99]. O carregamento sombra garante que as atualizações dos valores de comparação ocorram de forma síncrona com os eventos do temporizador, evitando glitches nas formas de onda PWM[cite: 100].
* **Sintaxe**[cite: 101]:
    ```c
    void EPWM_setCounterCompareShadowLoadMode(uint32_t base,
                                              EPWM_CounterCompareModule compModule,
                                              EPWM_LoadMode loadMode);
    ```
* **Parâmetros**:
    * `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`)[cite: 101].
    * `compModule`: O módulo de comparação a ser configurado[cite: 102]. Deve ser um dos valores do enum `EPWM_CounterCompareModule`[cite: 102]:
        * `EPWM_COUNTER_COMPARE_A`: Para o registrador de comparação A (`CMPA`)[cite: 102].
        * `EPWM_COUNTER_COMPARE_B`: Para o registrador de comparação B (`CMPB`)[cite: 103].
    * `loadMode`: O modo de carregamento sombra[cite: 103]. Deve ser um dos valores do enum `EPWM_LoadMode`[cite: 104]:
        * `EPWM_COMP_LOAD_ON_CNTR_ZERO`: Carrega o valor sombra quando o contador da base de tempo atinge zero[cite: 104].
        * `EPWM_COMP_LOAD_ON_CNTR_PRD`: Carrega o valor sombra quando o contador da base de tempo atinge o período[cite: 106].
        * `EPWM_COMP_LOAD_ON_CNTR_ZERO_OR_PRD`: Carrega o valor sombra quando o contador atinge zero ou o período[cite: 107].
        * `EPWM_COMP_LOAD_FREEZE`: O carregamento do valor sombra é congelado[cite: 108].
* **Valor de Retorno**: Nenhum (função void)[cite: 108].
* **Descrição**: Esta função configura os bits `CMPC` e `CMPB` no registrador `CMPCTL` do EPWM[cite: 109]. Ao utilizar o carregamento sombra, você garante que as alterações nos valores de comparação não afetem a forma de onda PWM imediatamente, mas sim no próximo evento de carregamento especificado (ZERO, PRD ou ambos)[cite: 110]. Isso é crucial para manter a integridade da forma de onda e evitar transições indesejadas[cite: 111].
* **Exemplo**[cite: 112]:
    ```c
    // Configura o CMPA do EPWM1 para carregar seu valor sombra no evento zero do contador
    EPWM_setCounterCompareShadowLoadMode(EPWM1_BASE,
                                         EPWM_COUNTER_COMPARE_A,
                                         EPWM_COMP_LOAD_ON_CNTR_ZERO); [cite: 112]
    // Configura o CMPB do EPWM2 para carregar seu valor sombra no evento de período do contador
    EPWM_setCounterCompareShadowLoadMode(EPWM2_BASE,
                                         EPWM_COUNTER_COMPARE_B,
                                         EPWM_COMP_LOAD_ON_CNTR_PRD); [cite: 113]
    ```

### 2.8 `EPWM_setCounterCompareValue()`

* **Propósito**: Define o valor de comparação para um dos registradores de comparação (`CMPA` ou `CMPB`) do módulo EPWM[cite: 114]. Esses valores são cruciais para determinar o ciclo de trabalho e as transições de borda dos sinais PWM gerados[cite: 115].
* **Sintaxe**[cite: 116]:
    ```c
    void EPWM_setCounterCompareValue(uint32_t base,
                                     EPWM_CounterCompareModule compModule,
                                     uint16_t compareValue);
    ```
* **Parâmetros**:
    * `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`)[cite: 117].
    * `compModule`: O módulo de comparação a ser configurado[cite: 118]. Deve ser um dos valores do enum `EPWM_CounterCompareModule`[cite: 118]:
        * `EPWM_COUNTER_COMPARE_A`: Para o registrador de comparação A (`CMPA`)[cite: 118].
        * `EPWM_COUNTER_COMPARE_B`: Para o registrador de comparação B (`CMPB`)[cite: 119].
    * `compareValue`: O valor a ser carregado no registrador de comparação[cite: 119]. Este valor é comparado com o contador da base de tempo (`TBCTR`) para gerar eventos de acionamento[cite: 120]. O valor máximo é 0xFFFF (65535)[cite: 121].
* **Valor de Retorno**: Nenhum (função void)[cite: 121].
* **Descrição**: Esta função escreve o `compareValue` fornecido no registrador `CMPA` ou `CMPB` do EPWM, dependendo do `compModule` selecionado[cite: 122]. Os valores de comparação são usados pelos qualificadores de ação para determinar quando a saída PWM deve mudar de estado (por exemplo, de alta para baixa ou vice-versa)[cite: 123]. A relação entre o valor de comparação e o período da base de tempo (`TBPRD`) define o ciclo de trabalho do sinal PWM[cite: 124]. É importante notar que, se o carregamento sombra estiver habilitado para o módulo de comparação, o `compareValue` será escrito no registrador sombra e só será transferido para o registrador ativo no próximo evento de carregamento sombra[cite: 125].
* **Exemplo**[cite: 126]:
    ```c
    // Define o valor de comparação A do EPWM1 para 500
    EPWM_setCounterCompareValue(EPWM1_BASE, EPWM_COUNTER_COMPARE_A, 500U); [cite: 126]
    // Define o valor de comparação B do EPWM2 para 250
    EPWM_setCounterCompareValue(EPWM2_BASE, EPWM_COUNTER_COMPARE_B, 250U); [cite: 127]
    ```

### 2.9 `EPWM_setActionQualifierAction()`

* **Propósito**: Configura a ação que ocorre nas saídas `EPWMxA` ou `EPWMxB` em resposta a eventos específicos do contador da base de tempo ou de comparação[cite: 128]. Esta é a função central para definir como a forma de onda PWM é gerada[cite: 129].
* **Sintaxe**[cite: 130]:
    ```c
    void EPWM_setActionQualifierAction(uint32_t base,
                                       EPWM_ActionQualifierOutput output,
                                       EPWM_ActionQualifierOutputAction action,
                                       EPWM_ActionQualifierEvent event);
    ```
* **Parâmetros**[cite: 131]:
    * `base`: O endereço base do módulo EPWM (e.g., `EPWM1_BASE`)[cite: 131].
    * `output`: A saída EPWM a ser configurada[cite: 131]. Deve ser um dos valores do enum `EPWM_ActionQualifierOutput`[cite: 132]:
        * `EPWM_AQ_OUTPUT_A`: Para a saída `EPWMxA`[cite: 132].
        * `EPWM_AQ_OUTPUT_B`: Para a saída `EPWMxB`[cite: 132].
    * `action`: A ação a ser tomada na saída quando o evento ocorrer[cite: 133]. Deve ser um dos valores do enum `EPWM_ActionQualifierOutputAction`[cite: 134]:
        * `EPWM_AQ_OUTPUT_NO_CHANGE`: Nenhuma mudança no estado da saída[cite: 134].
        * `EPWM_AQ_OUTPUT_LOW`: Força a saída para o estado baixo[cite: 135].
        * `EPWM_AQ_OUTPUT_HIGH`: Força a saída para o estado alto[cite: 135].
        * `EPWM_AQ_OUTPUT_TOGGLE`: Inverte o estado atual da saída[cite: 136].
    * `event`: O evento que aciona a ação[cite: 136]. Deve ser um dos valores do enum `EPWM_ActionQualifierEvent`[cite: 137]:
        * `EPWM_AQ_OUTPUT_ON_TIMEBASE_ZERO`: Ação no evento de contador atingir zero[cite: 137].
        * `EPWM_AQ_OUTPUT_ON_TIMEBASE_PERIOD`: Ação no evento de contador atingir o período[cite: 138].
        * `EPWM_AQ_OUTPUT_ON_TIMEBASE_UP_CMPA`: Ação quando o contador está crescendo e atinge `CMPA`[cite: 138].
        * `EPWM_AQ_OUTPUT_ON_TIMEBASE_DOWN_CMPA`: Ação quando o contador está decrescendo e atinge `CMPA`[cite: 139].
        * `EPWM_AQ_OUTPUT_ON_TIMEBASE_UP_CMPB`: Ação quando o contador está crescendo e atinge `CMPB`[cite: 139].
        * `EPWM_AQ_OUTPUT_ON_TIMEBASE_DOWN_CMPB`: Ação quando o contador está decrescendo e atinge `CMPB`[cite: 140].
* **Valor de Retorno**: Nenhum (função void)[cite: 140].
* **Descrição**: Esta função configura os bits de ação qualificador nos registradores `AQCTLA` e `AQCTLB` do EPWM[cite: 141, 142]. Ela permite que você defina com precisão quando e como as saídas PWM (`EPWMxA` e `EPWMxB`) devem mudar de estado[cite: 143]. Por exemplo, você pode configurar `EPWMxA` para ir para ALTO no evento `ZERO` do contador e para BAIXO no evento `UP_CMPA`, criando uma forma de onda PWM básica[cite: 144]. A combinação de diferentes ações e eventos permite a criação de formas de onda complexas e controladas com precisão[cite: 145].
* **Exemplo**[cite: 146]:
    ```c
    // Configura EPWM1A para ir para ALTO quando o contador atinge zero
    EPWM_setActionQualifierAction(EPWM1_BASE,
                                  EPWM_AQ_OUTPUT_A,
                                  EPWM_AQ_OUTPUT_HIGH,
                                  EPWM_AQ_OUTPUT_ON_TIMEBASE_ZERO); [cite: 146]

    // Configura EPWM1A para ir para BAIXO quando o contador está crescendo e atinge CMPA
    EPWM_setActionQualifierAction(EPWM1_BASE,
                                  EPWM_AQ_OUTPUT_A,
                                  EPWM_AQ_OUTPUT_LOW,
                                  EPWM_AQ_OUTPUT_ON_TIMEBASE_UP_CMPA); [cite: 147]

    // Configura EPWM1B para alternar seu estado no evento de período
    EPWM_setActionQualifierAction(EPWM1_BASE,
                                  EPWM_AQ_OUTPUT_B,
                                  EPWM_AQ_OUTPUT_TOGGLE,
                                  EPWM_AQ_OUTPUT_ON_TIMEBASE_PERIOD); [cite: 148]
    ```

### 2.10 `EPWM_isBaseValid()`

* **Propósito**: Verifica se o endereço base fornecido para um módulo EPWM é válido[cite: 149]. Esta função é tipicamente usada internamente pelos drivers para garantir que as operações de registro sejam realizadas em endereços de memória válidos, prevenindo acessos inválidos e comportamentos inesperados[cite: 150].
* **Sintaxe**[cite: 151]:
    ```c
    bool EPWM_isBaseValid(uint32_t base);
    ```
* **Parâmetros**:
    * `base`: O endereço base do módulo EPWM a ser validado (e.g., `EPWM1_BASE`, `EPWM2_BASE`)[cite: 151].
* **Valor de Retorno**: `true` se o endereço base for válido; `false` caso contrário[cite: 152, 153].
* **Descrição**: Embora a implementação exata desta função não esteja presente no arquivo epwm.c fornecido, sua presença em macros `ASSERT()` sugere que ela é uma função de utilidade que valida se o endereço base corresponde a um dos endereços base conhecidos e válidos para os módulos EPWM no microcontrolador DSP28004x[cite: 153]. Isso é uma prática comum em bibliotecas de drivers para garantir a robustez e a segurança do código, especialmente em sistemas embarcados onde acessos de memória inválidos podem levar a falhas críticas[cite: 154].
* **Exemplo**[cite: 155]:
    ```c
    // Exemplo de uso interno (como visto em outras funções do driver)
    // ASSERT(EPWM_isBaseValid(base)); [cite: 155]

    // Exemplo de uso em código de aplicação para validação
    if (EPWM_isBaseValid(EPWM3_BASE)) [cite: 156]
    {
        // O endereço base do EPWM3 é válido, prossiga com a configuração [cite: 156]
        // EPWM_setEmulationMode(EPWM3_BASE, EPWM_EMULATION_FREE_RUN); [cite: 156]
    }
    else
    {
        // O endereço base não é válido, tratar o erro [cite: 157]
        // Logar erro, retornar, etc. [cite: 157]
    }
    ```
