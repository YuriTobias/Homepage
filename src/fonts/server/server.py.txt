import os
import time
import socket
import logging

SEG_SIZE = 1024
TIMEOUT = 4

# Configure logging
logging.basicConfig(
    filename="nocheck_udp_server_log.log",  # Log file name
    level=logging.INFO,         # Log level
    format="%(asctime)s - %(levelname)s - %(message)s"
)


def tcp_server(host, port):
    os.makedirs("received_files", exist_ok=True)  # Ensure the directory exists

    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server_socket:
        server_socket.bind((host, port))
        server_socket.listen(1)
        print(f"TCP Server listening on {host}:{port}")

        logging.info(f"TCP Server listening on {host}:{port}")
        conn, addr = server_socket.accept()
        with conn:
            print(f"Connected by {addr}")
            logging.info(f"Connected by {addr}.")

            start_timestamp = conn.recv(SEG_SIZE).decode()
            start_timestamp = float(start_timestamp)
            
            start_time = time.time()
            logging.info(f"Server process started at {start_time}.")
            logging.info(f"Transmission started at {start_timestamp} accordingdly to client")

            # Receive the padded file name
            file_name = conn.recv(SEG_SIZE).decode()
            file_path = os.path.join("received_files", file_name)
            print(f"Receiving file '{file_name}'...")
            logging.info(f"Receiving file '{file_name}'.")

            with open(file_path, 'wb') as f:
                while True:
                    data = conn.recv(SEG_SIZE)
                    if not data:  # End of file
                        break
                    f.write(data)

            print(f"File '{file_name}' received and saved at '{file_path}'.")
            logging.info(f"File '{file_name}' received and saved at '{file_path}'.")
            end_time = time.time()
            logging.info(f"Transmission and server process ended at {end_time}.")
            logging.info(f"Total transmission time: {end_time - start_timestamp - 0.2:.10f} seconds.") # Removes 0.2 because of the client sleep
    

def udp_server(host, port):
    os.makedirs("received_files", exist_ok=True)  # Ensure the directory exists

    with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as server_socket:
        server_socket.bind((host, port))
        print(f"UDP Server listening on {host}:{port}")

        logging.info(f"UDP Server listening on {host}:{port}")

        try:
            # Receive the start timestamp
            start_timestamp, addr = server_socket.recvfrom(SEG_SIZE)  # Receive from the UDP socket
            start_timestamp = start_timestamp.decode()  # Decode the received bytes
            start_timestamp = float(start_timestamp)    # Convert the decoded string to a float

            start_time = time.time()
            logging.info(f"Server process started at {start_time}.")
            logging.info(f"Transmission started at {start_timestamp} accordingdly to client")

            # Set the timeout for receiving data
            server_socket.settimeout(TIMEOUT)

            # Receive the padded file name
            file_name, addr = server_socket.recvfrom(SEG_SIZE)
            file_name = file_name.decode()
            file_path = os.path.join("received_files", file_name)
            
            print(f"Receiving file '{file_name}' from {addr}...")
            logging.info(f"Receiving file '{file_name}'.")
            
            timeout = 0
            with open(file_path, 'wb') as f:
                while True:
                    try:
                        data, addr = server_socket.recvfrom(SEG_SIZE)
                        if data == b"EOF":  # End of File marker
                            break
                        f.write(data)
                    except socket.timeout:
                        # Timeout occurred; send a request to the client for EOF
                        timeout = 1
                        print("Timeout reached. EOF segment lost...")
                        logging.info(f"Timeout reached. EOF segment lost...")
                        break

            # Get the file size and log it
            file_size = os.path.getsize(file_path)
            print(f"File '{file_name}' received successfully and saved at '{file_path}'. Size: {file_size} bytes.")
            logging.info(f"File '{file_name}' received successfully and saved at '{file_path}'. Size: {file_size} bytes.")

            end_time = time.time()
            logging.info(f"Transmission and server process ended at {end_time}.")
            if timeout == 0:
                logging.info(f"Total transmission time: {end_time - start_timestamp:.10f} seconds.")
            else:
                logging.info(f"Total transmission time: {end_time - start_timestamp - 4:.10f} seconds.")
        except socket.timeout:
            print("Initial file name reception timed out. Closing server.")
            logging.info(f"Initial file name reception timed out. Closing server.")
        except Exception as e:
            print(f"An error occurred: {e}")

        

def main():
    # List of server types
    server_types = ["TCP", "UDP"]

    # Ask the user to choose a server type
    print("Choose the type of server:")
    for i, server_type in enumerate(server_types, start=1):
        print(f"{i}. {server_type}")

    choice = int(input("Enter your choice (1 or 2): "))
    if choice not in [1, 2]:
        print("Invalid choice. Exiting.")
        return

    # Get host and port from the user
    host = input("Enter the host (e.g., 127.0.0.1): ").strip()
    port = int(input("Enter the port (e.g., 12345): "))

    # Run the chosen server
    if choice == 1:
        print("Starting TCP server...")
        tcp_server(host, port)
    elif choice == 2:
        print("Starting UDP server...")
        udp_server(host, port)

if __name__ == "__main__":
    main()