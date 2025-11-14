# README – Solución Completa del Laboratorio Digitales III

**Autor:**  
**Fecha:**  

Este repositorio contiene el desarrollo completo de los 4 puntos del laboratorio:  
1. Análisis de sentimientos por imágenes  
2. ETL + Dashboard (Streamlit)  
3. Exploración tecnológica  
4. Proyecto para convocatoria MinCiencias  

---

# 🟦 PUNTO 1 — ANÁLISIS DE SENTIMIENTOS POR IMÁGENES  
## 🎯 Objetivo  
Detectar los sentimientos **feliz**, **bravo**, **triste** usando **MediaPipe**, **hilos**, **mutex** y **semaforización**, implementando un pipeline concurrente de procesamiento de imágenes.

---

## 📌 Descripción  
El sistema se divide en dos hilos principales:

- **Productor:** captura frames desde la cámara y los agrega a una cola protegida por **mutex**.  
- **Consumidor:** procesa frames usando **MediaPipe FaceMesh** para obtener landmarks y clasifica el sentimiento.  
- Se usa un **semáforo** para controlar el tamaño máximo del buffer y la sincronización.

---

## 🧠 Código Principal (`sentiment_detector.py`)

```python
import cv2
import mediapipe as mp
import threading
import queue
import time

# Cola compartida con máximo 5 elementos
frame_queue = queue.Queue(maxsize=5)

mutex = threading.Lock()
semaforo = threading.Semaphore(0)

mp_face = mp.solutions.face_mesh.FaceMesh(refine_landmarks=True)

def clasificar_emocion(landmarks):
    # Reglas simplificadas
    # (Esto se puede reemplazar por un modelo ML)
    boca = landmarks[13].y - landmarks[14].y
    cejas = landmarks[285].y - landmarks[55].y
    
    if boca < -0.02:
        return "Feliz"
    elif cejas < -0.03:
        return "Bravo"
    else:
        return "Triste"

def productor():
    cap = cv2.VideoCapture(0)
    while True:
        ret, frame = cap.read()
        if not ret:
            break

        if not frame_queue.full():
            mutex.acquire()
            frame_queue.put(frame)
            mutex.release()
            semaforo.release()

def consumidor():
    while True:
        semaforo.acquire()
        mutex.acquire()
        frame = frame_queue.get()
        mutex.release()

        rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        result = mp_face.process(rgb)

        if result.multi_face_landmarks:
            puntos = result.multi_face_landmarks[0].landmark
            emocion = clasificar_emocion(puntos)
            cv2.putText(frame, emocion, (30,50),
                        cv2.FONT_HERSHEY_SIMPLEX, 1, (0,255,0), 2)

        cv2.imshow("Detector Sentimientos", frame)
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

# Lanzar hilos
threading.Thread(target=productor).start()
threading.Thread(target=consumidor).start()
🖼️ Capturas del Punto 1
Colocar aquí las imágenes del detector funcionando



🟩 PUNTO 2 — ETL + DASHBOARD STREAMLIT
🎯 Objetivo
Crear un pipeline ETL sobre la base de datos del proyecto "Túnel carpiano" y construir un dashboard en Streamlit mostrando análisis, KPIs y gráficas.

📌 ETL (etl_pipeline.py)
python
Copiar código
import pandas as pd
from sqlalchemy import create_engine

def run_etl(db_uri):
    engine = create_engine(db_uri)

    df = pd.read_sql("SELECT * FROM sensores", engine)

    # ---- TRANSFORMACIONES ----
    df["fuerza_media"] = (df["fuerza1"] + df["fuerza2"] + df["fuerza3"]) / 3
    df = df.dropna()

    df.to_csv("./data/processed/etl_output.csv", index=False)
    print("ETL finalizado. Archivo generado: etl_output.csv")

if __name__ == "__main__":
    run_etl("sqlite:///data/tunel_carpiano.sqlite")
📊 DASHBOARD STREAMLIT (streamlit_app.py)
python
Copiar código
import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv("./data/processed/etl_output.csv")

st.title("Dashboard – Túnel Carpiano")

st.metric("Registros procesados", len(df))
st.metric("Fuerza media global", round(df["fuerza_media"].mean(), 2))

fig, ax = plt.subplots()
ax.plot(df["fuerza_media"])
st.pyplot(fig)
🖼️ Capturas del Punto 2


🟨 PUNTO 3 — EXPLORACIÓN TECNOLÓGICA
Tecnologías Exploradas
🅰 Terraform
Infraestructura como código.

hcl
Copiar código
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "vm1" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"
}
📸 Captura aquí

🅱 Ansible
Automatización de servidores.

yaml
Copiar código
- name: Instalar dependencias
  hosts: all
  tasks:
    - name: Actualizar paquetes
      apt:
        update_cache: yes
📸 Captura aquí

🅲 RabbitMQ
Mensajería entre servicios.

python
Copiar código
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='sensor')

channel.basic_publish(exchange='', routing_key='sensor', body="dato recibido")
connection.close()
📸 Captura aquí

🅳 OpenStack
Nube privada.
Componentes principales: Nova, Neutron, Glance, Keystone, Cinder.

📸 Captura aquí

🅴 Cuadrante de Gartner
Análisis comparativo de proveedores en IA y cloud (Leaders, Visionaries, Challengers).

📸 Captura aquí

🟧 PUNTO 4 — PROYECTO CONVOCATORIA MINCIENCIAS
🎯 Título
PNEEDIA — Plataforma Nacional Educativa y de Entrenamiento para IA

📌 Problema
Colombia no posee infraestructura propia para entrenamiento de modelos IA avanzados, lo que limita la soberanía tecnológica.

💡 Solución Propuesta
Diseño de una plataforma basada en OpenStack + GPU clusters + MLflow/Kubeflow para permitir:

Entrenamiento de IA nacional

MLOps estandarizado

Banco de datos federado

Servicios IA para universidades, empresas y gobierno

🧱 Arquitectura (colocar diagrama aquí)
css
Copiar código
[Edge Data] → [OpenStack Cloud] → [GPU Cluster] → [Kubeflow/MLflow] → [API Models] → [Usuarios]
Colocar diagrama aquí:
./docs/arquitectura.png

🔮 Tecnologías Futuras Recomendadas
TinyML

Federated Learning

LLM locales optimizados

ONNX + cuantización

Serving en Kubernetes

Ceph / Lustre para almacenamiento distribuido

📂 Estructura del repositorio
Copiar código
/
├── punto1_sentimientos/
├── punto2_etl_dashboard/
├── punto3_exploracion/
├── punto4_minciencias/
├── docs/
└── README.md
✔ Requisitos
nginx
Copiar código
mediapipe
numpy
opencv-python
streamlit
pandas
sqlalchemy
matplotlib
🖼️ Sección general de capturas
Espacio para imágenes globales del proyecto.

👤 Autor
[Tu Nombre]
Contacto: [tu correo]
