import socket
import time

ESP32_IP = "192.168.0.47"
PORT = 8080

def tcp_test(data_size, repetitions):
    data = "A" * data_size
    total_time = 0

    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as client:
        client.connect((ESP32_IP, PORT))
        for _ in range(repetitions):
            start_time = time.time()
            client.sendall(data.encode())
            response = client.recv(1024)
            total_time += (time.time() - start_time)
            print(f"Resposta: {response.decode().strip()}")

    avg_time = total_time / repetitions
    print(f"Tamanho do Pacote: {data_size} bytes | RTT Médio: {avg_time:.6f} segundos")

# Testa pacotes de diferentes tamanhos
for size in [1450]:
    tcp_test(size, 5)
