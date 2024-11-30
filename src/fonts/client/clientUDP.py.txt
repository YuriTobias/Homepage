import socket
from datetime import datetime
import time

ESP32_IP = "192.168.0.47"
PORT = 8080

def udp_test(data_size, repetitions):
    data = "A" * data_size
    total_time = 0
    print(f"Enviando pacotes de {data_size} bytes para {ESP32_IP}:{PORT}")

    with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as client:
        print("Conectado ao servidor")
        for _ in range(repetitions):
            timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
            print(f"{timestamp}: Enviando pacote de {_ + 1} de {repetitions}")

            client.sendto(data.encode(), (ESP32_IP, PORT))
            
for size in [1450]:
    udp_test(size, 10000)
