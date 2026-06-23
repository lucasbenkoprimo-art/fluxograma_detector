```mermaid
flowchart TD
    %% Estilos
    classDef inicio_fim fill:#f9f,stroke:#333,stroke-width:2px;
    classDef processo fill:#bbf,stroke:#333,stroke-width:1px;
    classDef decisao fill:#ffb,stroke:#333,stroke-width:1px;
    classDef alerta fill:#fbb,stroke:#333,stroke-width:2px;

    %% Setup
    Start([Inicio Ligar Arduino]) --> Setup[Abre Serial e Conecta ao Blynk]
    Setup --> Calibra[Le NTC inicial e define Pico Recente]
    Calibra --> LoopStart[Inicio do LOOP]

    %% Loop Geral
    LoopStart --> BlynkRun[Blynk.run]
    BlynkRun --> LeNTC[Le ADC e calcula Temperatura Atual]
    
    %% Bloco de Rastreamento de Pico
    LeNTC --> Decisao1{Temp Atual maior que Pico Recente?}
    
    Decisao1 -- Sim --> AtualizaPico[Pico Recente igual Temp Atual e Reseta timer da respiracao]
    AtualizaPico --> DecisaoAutoCura{Alerta esta ativo?}
    DecisaoAutoCura -- Sim --> AutoCura[Alerta igual 0 e Limpa Alerta no Serial]
    DecisaoAutoCura -- Nao --> CalcQueda
    AutoCura --> CalcQueda
    
    Decisao1 -- Nao --> Decaimento[Decaimento lento do Pico]
    Decaimento --> CalcQueda

    %% Bloco de Queda e Print
    CalcQueda[Calcula Queda igual Pico menos Temp Atual]
    CalcQueda --> DecisaoPrint{Alerta igual 0?}
    DecisaoPrint -- Sim --> PrintSerial[Exibe dados no Serial]
    DecisaoPrint -- Nao --> DecisaoDisparo
    PrintSerial --> DecisaoDisparo

    %% Bloco de Alarme / Disparo
    DecisaoDisparo{Queda maior que 0.5 e Sem respirar maior que 10s?}
    DecisaoDisparo -- Sim --> DecisaoSpam{Alerta igual 0?}
    DecisaoSpam -- Sim --> DisparaAlerta[ALERTA APNEIA DETECTADA - Envia para o Blynk e Alerta igual 1]
    DecisaoSpam -- Nao --> DecisaoReset
    DisparaAlerta --> DecisaoReset
    DecisaoDisparo -- Nao --> DecisaoReset

    %% Bloco de Reset Ambiental
    DecisaoReset{Tempo do Quarto maior que 60s?}
    DecisaoReset -- Sim --> ResetAmbiente[Pico Recente igual Temp Atual e Reseta timer ambiente]
    DecisaoReset -- Nao --> DelayFim
    ResetAmbiente --> DelayFim

    %% Fim do Ciclo
    DelayFim[Delay 150ms] --> LoopStart

    %% Aplicacao dos Estilos nas Caixas
    class Start,LoopStart inicio_fim;
    class Decisao1,DecisaoAutoCura,DecisaoPrint,DecisaoDisparo,DecisaoSpam,DecisaoReset decisao;
    class DisparaAlerta alerta;
