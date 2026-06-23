```mermaid
flowchart TD
    %% Estilos
    classDef inicio_fim fill:#f9f,stroke:#333,stroke-width:2px;
    classDef processo fill:#bbf,stroke:#333,stroke-width:1px;
    classDef decisao fill:#ffb,stroke:#333,stroke-width:1px;
    classDef alerta fill:#fbb,stroke:#333,stroke-width:2px;

    %% Setup
    Start([Início: Ligar Arduino]) --> Setup[Abre Serial & Conecta ao Blynk]
    Setup --> Calibra[Lê NTC inicial e define Pico Recente]
    Calibra --> LoopStart[Início do LOOP]

    %% Loop Geral
    LoopStart --> BlynkRun[Blynk.run]
    BlynkRun --> LeNTC["Lê ADC e calcula Temperatura Atual °C"]
    
    %% Bloco de Rastreamento de Pico
    LeNTC --> Decisao1{Temp Atual > Pico Recente?}
    
    Decisao1 -- Sim --> AtualizaPico["Pico Recente = Temp Atual<br>Reseta timer da respiração"]
    AtualizaPico --> DecisaoAutoCura{Alerta está ativo?}
    DecisaoAutoCura -- Sim --> AutoCura["Alerta = 0<br>Serial: Respiração Normalizada"]
    DecisaoAutoCura -- Não --> CalcQueda
    AutoCura --> CalcQueda
    
    Decisao1 -- Não --> Decaimento["Decaimento lento do Pico<br>Pico = Pico - 0.002"]
    Decaimento --> CalcQueda

    %% Bloco de Queda e Print
    CalcQueda[Calcula Queda = Pico - Temp Atual]
    CalcQueda --> DecisaoPrint{Alerta == 0?}
    DecisaoPrint -- Sim --> PrintSerial[Exibe Temp, Pico e Queda no Serial]
    DecisaoPrint -- Não --> DecisaoDisparo
    PrintSerial --> DecisaoDisparo

    %% Bloco de Alarme / Disparo
    DecisaoDisparo["Queda > 0.5 E<br>Sem respirar > 10s?"]
    DecisaoDisparo -- Sim --> DecisaoSpam{Alerta == 0?}
    DecisaoSpam -- Sim --> DisparaAlerta:::alerta["Serial: APNEIA DETECTADA<br>Blynk: Envia Notificação<br>Alerta = 1"]
    DecisaoSpam -- Não --> DecisaoReset
    DisparaAlerta --> DecisaoReset
    DecisaoDisparo -- Não --> DecisaoReset

    %% Bloco de Reset Ambiental
    DecisaoReset{Tempo do Quarto > 60s?}
    DecisaoReset -- Sim --> ResetAmbiente["Pico Recente = Temp Atual<br>Reseta timer do ambiente"]
    DecisaoReset -- Não --> DelayFim
    ResetAmbiente --> DelayFim

    %% Fim do Ciclo
    DelayFim[Delay 150ms] --> LoopStart

    class Start,LoopStart inicio_fim;
    class Decisao1,DecisaoAutoCura,DecisaoPrint,DecisaoDisparo,DecisaoSpam,DecisaoReset decisao;
