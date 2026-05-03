graph TD
    %% Power Subsystem
    subgraph Power Subsystem [Power & Supervisory Subsystem]
        USB[USB-C 5V Input] -->|Charge| TP4056[TP4056 Charge Controller]
        BATT[3.7V 300mAh LiPo] <-->|Charge/Discharge| TP4056
        TP4056 -->|V_OUT| SW[SPDT Isolation Switch]
        SW -->|System VCC 3.2V-4.2V| VCC_BUS((VCC BUS))
        VCC_BUS -->|VDD| TLV[TLV809L30 Brown-Out Lockout]
        TLV -->|Active Low RESET| R_ISO[1kΩ ISP Isolation Resistor]
    end

    %% MCU Subsystem
    subgraph MCU Subsystem [Processing & ADC Subsystem]
        R_ISO -->|RESET Pin 4| MCU[ATtiny84 Microcontroller]
        VCC_BUS -->|Pin 1| MCU
        
        %% EMI Filtered NTC Array
        VCC_BUS -->|Pull-up| R_NTC1[10kΩ Fixed] --> ADC1[PA0 / Pin 13]
        NTC1[10kΩ NTC Thermistor 1] -->|To GND| ADC1
        CAP1[100nF Ceramic] -->|Low-Pass Filter to GND| ADC1
    end

    %% Drive Subsystem
    subgraph Actuation Subsystem [Actuation & Hardware Safety Loop]
        VCC_BUS -->|VM / VCC| DRV[DRV8833 Dual H-Bridge]
        CAP_BULK[1000µF Electrolytic] -->|Transient Suppression| DRV
        MCU -->|122Hz PWM| DRV_IN[AIN1 & BIN1 Tied]
        DRV_IN --> DRV
        
        %% Redundant Safety Loop (Single Motor Example)
        DRV -->|Channel Output| R_LIMIT[0.5Ω Inrush Resistor]
        R_LIMIT -->|Series Connection| KSD[KSD-01F 40°C Bimetal Switch]
        KSD -->|Hardware Cutoff| MOT[10mm ERM Vibration Motor]
    end

    %% Grounding
    MOT --> GND((Star GND))
    MCU --> GND
    DRV --> GND
    TP4056 --> GND
