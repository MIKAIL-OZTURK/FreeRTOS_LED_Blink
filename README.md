# FreeRTOS_LED_Blink
Bu proje, **STM32F407VG Discovery Board** üzerinde **FreeRTOS** kullanılarak geliştirilmiştir. Projede, iki farklı LED (PD12 ve PD14 pinlerinde) ayrı görevler (tasks) içerisinde belirli aralıklarla yakılıp 
söndürülmektedir.


## 🔧 Proje Özeti
- Donanım: STM32F407VG Discovery Kit
- IDE: STM32CubeIDE v1.16.1
- RTOS: FreeRTOS (Classic API - `CMSIS_V1` değil!)

## 📁 İçerik

- [Görev Oluşturma](#1--görev-oluşturma)
- [Görev Fonksiyonları](#2--görev-fonksiyonları)
- [Zamanlayıcıyı Başlatma](#3--zamanlayıcıyı-başlatma)
- [Örnek Kod](#4--örnek-kod-basitleştirilmiş)
- [Uygulama Videosu](#uygulama-videosu)

## 1. Görev Oluşturma

Görevler `xTaskCreate()` fonksiyonu ile tanımlanır.

```c
xTaskCreate(LedTask1, "Led_Task_1", 128, NULL, 1, NULL); 
xTaskCreate(LedTask2, "Led_Task_2", 128, NULL, 1, NULL); 
```

| Parametre    | Açıklama                       |
| ------------ | ------------------------------ |
| Fonksiyon    | `LedTask1`, `LedTask2`         |
| İsim         | `"Led_Task_1"`, `"Led_Task_2"` |
| Stack Boyutu | `128`                          |
| Parametre    | `NULL`                         |
| Öncelik      | `1`                            |
| Task Handle  | `NULL`                         |


### ✅ Hata Kontrolü
Görevlerin başarıyla oluşturulup oluşturulmadığını kontrol etmek iyi bir uygulamadır:

```c
BaseType_t status;
status = xTaskCreate(LedTask1, "Led_Task_1", 128, NULL, 1, NULL);
if (status != pdPASS) {
    Error_Handler();  // Görev oluşturulamadı
}
```

## 2. Görev Fonksiyonları
Her görev, 1 saniyede bir ilgili pini toggle eder. Bu işlemler FreeRTOS görev döngüsünde yapılır.

```c
void LedTask1(void* pvParameters)
{
	while(1)
	{
		HAL_GPIO_TogglePin(GPIOD, GPIO_PIN_12);     
		vTaskDelay(pdMS_TO_TICKS(1000));            
	}
}

void LedTask2(void* pvParameters)
{
	while(1)
	{
		HAL_GPIO_TogglePin(GPIOD, GPIO_PIN_14);     
		vTaskDelay(pdMS_TO_TICKS(1000));            
	}
}

```

| 📌 pdMS_TO_TICKS(1000) ifadesi, 1000 milisaniyeyi tick süresine çevirir. FreeRTOS zaman tabanı tick cinsinden çalıştığı için bu dönüşüm önemlidir.




## 3. Zamanlayıcıyı Başlatma
Tüm görevler oluşturulduktan sonra, zamanlayıcı (Scheduler) başlatılır:

```c
vTaskStartScheduler();
```

Bu fonksiyon çağrıldıktan sonra, kontrol artık FreeRTOS'a geçer ve main() içindeki sonsuz döngüye girilmez.


## 4. Örnek Kod (Basitleştirilmiş)

```c
#include "main.h"
#include "FreeRTOS.h"
#include "task.h"

void LedTask1(void* pvParameters); 
void LedTask2(void* pvParameters); 

int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();

  xTaskCreate(LedTask1, "Led_Task_1", 128, NULL, 1, NULL); 
  xTaskCreate(LedTask2, "Led_Task_2", 128, NULL, 1, NULL); 

  vTaskStartScheduler();

  while (1) {}  // Scheduler başlatıldığı için buraya girilmez
}

void LedTask1(void* pvParameters)
{
	while(1)
	{
		HAL_GPIO_TogglePin(GPIOD, GPIO_PIN_12);     
		vTaskDelay(pdMS_TO_TICKS(1000));            
	}
}

void LedTask2(void* pvParameters)
{
	while(1)
	{
		HAL_GPIO_TogglePin(GPIOD, GPIO_PIN_14);     
		vTaskDelay(pdMS_TO_TICKS(1000));            
	}
}

```

## Uygulama Videosu



https://github.com/user-attachments/assets/94720553-4990-4f5f-9dea-e55ca97730a3



