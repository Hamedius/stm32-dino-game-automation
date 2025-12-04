# STM32 Dino Game Automation

This repository contains the **STM32-based Dino Game Automation** project.  
An STM32 Nucleo board reads a photoresistor (LDR) pointed at the game screen and uses a servo motor to physically press the keyboard when an obstacle (cactus) is detected, automatically playing the Chrome Dino game.

The design and logic are based on the project documentation **Dinasor.pdf**.

---

## 🎮 Project Idea

The goal of this project is to:

- Detect obstacles in the Dino game using a **light sensor**
- Convert that information into a **PWM control signal**
- Drive a **servo motor** that presses the *jump* key at the correct time

This demonstrates:

- ADC usage on STM32 (LDR as analog sensor)
- Timer + PWM for servo control
- Real‑time decision making based on sensor threshold
- Simple mechatronic system: **screen → sensor → MCU → servo → keyboard**

---

## 🧩 Hardware

Typical setup:

- **STM32 Nucleo-F411RE** (or similar STM32F4 board)
- **Photoresistor (LDR)** + resistor divider connected to **PA0 (ADC1_IN0)**
- **SG90 micro servo** connected to **PA8 (TIM1_CH1)**
- **On-board LED** on **PA5** used as debug indicator
- USB cable for power and programming

The LDR is placed facing the region of the screen where obstacles appear.  
When a cactus passes in front, the brightness changes → ADC value changes → MCU detects it and triggers a jump.

---

## 📁 Repository Structure

```text
stm32-dino-game-automation/
├── README.md
├── docs/
│   └── Dinasor.pdf
├── src/
    └── main.c
```

- `README.md` – this documentation file  
- `docs/Dinasor.pdf` – original project report  
- `src/main.c` – STM32Cube HAL firmware implementing the control logic  

---

## 🧠 Firmware Logic (Summary)

Core loop reads the ADC and controls the servo:

```c
HAL_ADC_Start(&hadc1);
HAL_ADC_PollForConversion(&hadc1, 10);
lux = HAL_ADC_GetValue(&hadc1);

if (lux < 2300) {
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
    __HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, 650);
    HAL_Delay(50);
} else {
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
    __HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, 560);
}

HAL_Delay(20);
```

- `lux` is the ADC reading from the LDR on **PA0**  
- Threshold `2300` separates “no cactus” vs “cactus detected”  
- PWM values `650` and `560` control servo angles (press vs rest)  
- LED on **PA5** turns ON while jump is triggered  

The rest of `main.c` (clock, ADC, TIM1, GPIO init) is already set up for you.

---

## ▶️ How to Use (STM32CubeIDE)

1. **Create a new STM32CubeIDE project** for your Nucleo board (e.g., Nucleo-F411RE).  
2. In that project, replace the generated `Core/Src/main.c` with the `src/main.c` from this repo.
3. Make sure `main.h` and other auto-generated files are kept from CubeMX.
4. Build the project.
5. Flash the firmware to the board.
6. Position:
   - LDR towards the screen area where obstacles appear
   - Servo arm above the keyboard *space* key (or game controller button)
7. Start the Chrome Dino game → the Dino should start jumping automatically when cacti appear.

If you need to tune performance, adjust:

- ADC threshold (`2300`)
- Servo PWM compare values (`560`, `650`)
- Delays (`50 ms`, `20 ms`)

---

## 🧪 Possible Improvements

- Adaptive threshold (auto-calibrate based on average light level)  
- Use an **HID keyboard emulator** instead of mechanical pressing  
- Add a **manual/auto mode switch** via a button  
- Log ADC values and timing via UART for analysis  
- Replace LDR with a proper image sensor or digital detector

---

## 👤 Authors

- **Hamed Nahvi**
- **Yasin Shadrouh**

GitHub: https://github.com/Hamedius

---

This repository is designed to be **clean and CV‑friendly**, highlighting your skills in:

- STM32 embedded development
- HAL drivers (ADC, TIM, GPIO)
- Real‑time control logic
- Mechatronics / hardware–software integration
