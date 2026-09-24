# Ejercicios Prácticos: Edge Computing, IoT y Sistemas Operativos

## 1. Edge Computing

### Ejercicio 1.1: Implementación de un Sistema de Procesamiento en Edge
**Objetivo**: Crear un sistema que procese datos de sensores en el edge antes de enviarlos a la nube.

**Solución**:

```python
import time
import json
import random
from collections import deque
from statistics import mean

class EdgeDevice:
    def __init__(self, device_id, buffer_size=60):
        self.device_id = device_id
        self.data_buffer = deque(maxlen=buffer_size)
        self.threshold = 25  # Umbral para alertas

    def collect_sensor_data(self):
        # Simula la recolección de datos de temperatura
        return {
            'temperature': random.uniform(20, 30),
            'humidity': random.uniform(40, 60),
            'timestamp': time.time()
        }

    def process_data(self, data):
        # Procesamiento en el edge
        self.data_buffer.append(data['temperature'])
        
        # Calcular promedio móvil
        avg_temp = mean(self.data_buffer)
        
        processed_data = {
            'device_id': self.device_id,
            'raw_temperature': data['temperature'],
            'average_temperature': avg_temp,
            'humidity': data['humidity'],
            'alert': avg_temp > self.threshold,
            'timestamp': data['timestamp']
        }
        
        return processed_data

    def should_send_to_cloud(self, processed_data):
        # Lógica para decidir si enviar a la nube
        return (processed_data['alert'] or 
                len(self.data_buffer) >= self.data_buffer.maxlen)

    def run(self):
        while True:
            # Recolectar datos
            raw_data = self.collect_sensor_data()
            
            # Procesar en el edge
            processed_data = self.process_data(raw_data)
            
            # Decidir si enviar a la nube
            if self.should_send_to_cloud(processed_data):
                self.send_to_cloud(processed_data)
            
            time.sleep(1)

    def send_to_cloud(self, data):
        # Simulación de envío a la nube
        print(f"Enviando a la nube: {json.dumps(data, indent=2)}")

# Uso del sistema
edge_device = EdgeDevice("sensor-001")
edge_device.run()
```

### Ejercicio 1.2: Sistema de Inferencia de ML en el Edge
**Objetivo**: Implementar un sistema de inferencia de machine learning en el edge para clasificación de imágenes.

**Solución**:

```python
import numpy as np
import cv2
from tensorflow.lite import Interpreter

class EdgeMLInference:
    def __init__(self, model_path):
        self.interpreter = Interpreter(model_path=model_path)
        self.interpreter.allocate_tensors()
        
        # Obtener detalles del modelo
        self.input_details = self.interpreter.get_input_details()
        self.output_details = self.interpreter.get_output_details()
        
        # Dimensiones de entrada requeridas
        self.height = self.input_details[0]['shape'][1]
        self.width = self.input_details[0]['shape'][2]

    def preprocess_image(self, image):
        # Redimensionar y preparar la imagen
        image = cv2.resize(image, (self.width, self.height))
        image = image.astype(np.float32)
        image = (image - 127.5) / 127.5
        return image

    def inference(self, image):
        # Preprocesar imagen
        processed_image = self.preprocess_image(image)
        
        # Preparar el tensor de entrada
        input_tensor = np.expand_dims(processed_image, axis=0)
        self.interpreter.set_tensor(
            self.input_details[0]['index'], 
            input_tensor
        )
        
        # Realizar inferencia
        self.interpreter.invoke()
        
        # Obtener resultados
        output_data = self.interpreter.get_tensor(
            self.output_details[0]['index']
        )
        return output_data[0]

    def post_process(self, predictions, labels_path):
        # Cargar etiquetas
        with open(labels_path, 'r') as f:
            labels = [line.strip() for line in f.readlines()]
        
        # Obtener top-5 predicciones
        top_k = predictions.argsort()[-5:][::-1]
        results = []
        
        for idx in top_k:
            results.append({
                'label': labels[idx],
                'confidence': float(predictions[idx])
            })
        
        return results

# Uso del sistema
def main():
    model_path = "model.tflite"
    labels_path = "labels.txt"
    
    # Inicializar sistema
    edge_ml = EdgeMLInference(model_path)
    
    # Capturar imagen de la cámara
    cap = cv2.VideoCapture(0)
    ret, frame = cap.read()
    cap.release()
    
    if ret:
        # Realizar inferencia
        predictions = edge_ml.inference(frame)
        
        # Post-procesar resultados
        results = edge_ml.post_process(predictions, labels_path)
        
        # Mostrar resultados
        for result in results:
            print(f"Clase: {result['label']}, "
                  f"Confianza: {result['confidence']:.2%}")

if __name__ == "__main__":
    main()
```

## 2. Sistemas Operativos para IoT

### Ejercicio 2.1: Implementación de un Sistema de Tiempo Real
**Objetivo**: Crear un sistema básico de tiempo real para control de dispositivos IoT.

**Solución** (usando FreeRTOS):

```c
#include "FreeRTOS.h"
#include "task.h"
#include "queue.h"

// Estructura para mensajes de sensores
typedef struct {
    uint8_t sensorId;
    float value;
    TickType_t timestamp;
} SensorData;

// Handles para colas y tareas
QueueHandle_t sensorQueue;
TaskHandle_t sensorTaskHandle;
TaskHandle_t processingTaskHandle;

// Tarea de lectura de sensor
void vSensorTask(void *pvParameters) {
    while (1) {
        SensorData data;
        data.sensorId = (uint8_t)pvParameters;
        data.value = readSensor(data.sensorId);  // Función hipotética
        data.timestamp = xTaskGetTickCount();
        
        // Enviar datos a la cola
        xQueueSend(sensorQueue, &data, portMAX_DELAY);
        
        // Esperar siguiente ciclo
        vTaskDelay(pdMS_TO_TICKS(100));
    }
}

// Tarea de procesamiento
void vProcessingTask(void *pvParameters) {
    SensorData data;
    
    while (1) {
        // Esperar datos del sensor
        if (xQueueReceive(sensorQueue, &data, portMAX_DELAY) == pdTRUE) {
            // Procesar datos
            processData(&data);  // Función hipotética
            
            // Verificar límites de tiempo
            TickType_t currentTime = xTaskGetTickCount();
            TickType_t processingTime = currentTime - data.timestamp;
            
            if (processingTime > pdMS_TO_TICKS(50)) {
                // Alerta de deadline perdido
                handleDeadlineMiss();  // Función hipotética
            }
        }
    }
}

// Inicialización del sistema
void initSystem(void) {
    // Crear cola para datos de sensores
    sensorQueue = xQueueCreate(10, sizeof(SensorData));
    
    // Crear tareas
    xTaskCreate(
        vSensorTask,
        "Sensor1",
        configMINIMAL_STACK_SIZE,
        (void *)1,
        tskIDLE_PRIORITY + 2,
        &sensorTaskHandle
    );
    
    xTaskCreate(
        vProcessingTask,
        "Processor",
        configMINIMAL_STACK_SIZE,
        NULL,
        tskIDLE_PRIORITY + 1,
        &processingTaskHandle
    );
    
    // Iniciar el scheduler
    vTaskStartScheduler();
}
```

### Ejercicio 2.2: Sistema de Gestión de Energía para IoT
**Objetivo**: Implementar un sistema de gestión de energía para dispositivos IoT.

**Solución**:

```c
#include <stdio.h>
#include <stdint.h>

// Estados de energía
typedef enum {
    POWER_ACTIVE,
    POWER_IDLE,
    POWER_SLEEP,
    POWER_DEEP_SLEEP
} PowerState;

// Configuración de energía
typedef struct {
    uint32_t voltage_mv;
    uint32_t current_ma;
    uint32_t battery_level;
    PowerState current_state;
    uint32_t sleep_timeout_ms;
} PowerConfig;

// Gestor de energía
typedef struct {
    PowerConfig config;
    uint32_t last_activity_time;
    uint32_t total_active_time;
    uint32_t total_sleep_time;
} PowerManager;

// Inicializar gestor de energía
void powerManager_init(PowerManager *pm) {
    pm->config.voltage_mv = 3300;
    pm->config.current_ma = 0;
    pm->config.battery_level = 100;
    pm->config.current_state = POWER_ACTIVE;
    pm->config.sleep_timeout_ms = 5000;
    
    pm->last_activity_time = 0;
    pm->total_active_time = 0;
    pm->total_sleep_time = 0;
}

// Actualizar estado de energía
void powerManager_update(PowerManager *pm, uint32_t current_time) {
    uint32_t time_since_activity = current_time - pm->last_activity_time;
    
    // Determinar estado basado en actividad
    if (time_since_activity > pm->config.sleep_timeout_ms) {
        if (pm->config.battery_level < 20) {
            powerManager_setState(pm, POWER_DEEP_SLEEP);
        } else if (time_since_activity > pm->config.sleep_timeout_ms * 2) {
            powerManager_setState(pm, POWER_SLEEP);
        } else {
            powerManager_setState(pm, POWER_IDLE);
        }
    }
    
    // Actualizar consumo de energía
    switch (pm->config.current_state) {
        case POWER_ACTIVE:
            pm->config.current_ma = 100;
            break;
        case POWER_IDLE:
            pm->config.current_ma = 50;
            break;
        case POWER_SLEEP:
            pm->config.current_ma = 10;
            break;
        case POWER_DEEP_SLEEP:
            pm->config.current_ma = 1;
            break;
    }
    
    // Actualizar nivel de batería
    updateBatteryLevel(pm);  // Función hipotética
}

// Registrar actividad
void powerManager_registerActivity(PowerManager *pm, uint32_t current_time) {
    pm->last_activity_time = current_time;
    
    if (pm->config.current_state != POWER_ACTIVE) {
        powerManager_setState(pm, POWER_ACTIVE);
    }
}

// Cambiar estado de energía
void powerManager_setState(PowerManager *pm, PowerState new_state) {
    if (pm->config.current_state != new_state) {
        // Realizar acciones específicas del cambio de estado
        switch (new_state) {
            case POWER_ACTIVE:
                enablePeripherals();  // Función hipotética
                break;
            case POWER_SLEEP:
                disableNonEssentialPeripherals();  // Función hipotética
                break;
            case POWER_DEEP_SLEEP:
                prepareForDeepSleep();  // Función hipotética
                break;
        }
        
        pm->config.current_state = new_state;
    }
}

// Ejemplo de uso
int main() {
    PowerManager pm;
    powerManager_init(&pm);
    
    // Simulación de ciclo principal
    uint32_t current_time = 0;
    while (1) {
        powerManager_update(&pm, current_time);
        
        // Simular alguna actividad
        if (checkForActivity()) {  // Función hipotética
            powerManager_registerActivity(&pm, current_time);
        }
        
        current_time += 100;  // Incremento de tiempo simulado
        delay(100);  // Función hipotética
    }
    
    return 0;
}
```

## 3. Discusión sobre el Futuro de los Sistemas Operativos y Cloud Computing

### Ejercicio 3.1: Implementación de un Sistema Híbrido Edge-Cloud
**Objetivo**: Crear un sistema que demuestre la integración entre edge computing y cloud computing.

**Solución**:

```python
import asyncio
import json
import aiohttp
from datetime import datetime
from typing import Dict, List

class HybridSystem:
    def __init__(self):
        self.edge_devices: Dict[str, EdgeDevice] = {}
        self.cloud_connection = CloudConnection()
        self.edge_cache = EdgeCache()
        
    async def process_data_hybrid(self, data: Dict):
        # Procesar en edge
        processed_data = await self.edge_cache.process(data)
        
        # Decidir dónde ejecutar
        if self.should_process_in_cloud(processed_data):
            # Enviar a la nube
            result = await self.cloud_connection.process(processed_data)
        else:
            # Procesar localmente
            result = await self.process_locally(processed_data)
            
        return result
        
    def should_process_in_cloud(self, data: Dict) -> bool:
        # Criterios para decidir:
        # 1. Complejidad del procesamiento
        # 2. Tamaño de datos
        # 3. Latencia requerida
        # 4. Disponibilidad de recursos locales
        
        if (data.get('complexity', 0) > 7 or
            len(str(data)) > 1000 or
            not self.edge_cache.has_resources()):
            return True
            
        return False
        
    async def process_locally(self, data: Dict):
        # Procesamiento local
        return await self.edge_cache.process_complete(data)

class EdgeCache:
    def __init__(self):
        self.cache = {}
        self.resources_available = True
        
    async def process(self, data: Dict):
        # Procesamiento inicial en edge
        result = {
            'timestamp': datetime.now().isoformat(),
            'device_id': data.get('device_id'),
            'processed':