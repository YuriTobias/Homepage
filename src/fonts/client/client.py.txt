import os
import time
import random
import string
import socket
import logging

SEG_SIZE = 1024

# Configure logging
logging.basicConfig(
    filename="nocheck_udp_client_log.log",  # Log file name
    level=logging.INFO,         # Log level
    format="%(asctime)s - %(levelname)s - %(message)s"
)

def tcp_client(host, port):
    # Ensure the directory exists
    os.makedirs("sent_files", exist_ok=True)

    # Ask the user for file details
    file_name = input("Enter the name of the file to use: ").strip()
    file_path = os.path.join("sent_files", file_name)
    file_size = os.path.getsize(file_path)
    
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as client_socket:
        # Disable checksum
        client_socket.setsockopt(socket.SOL_SOCKET, 11, 1)
        
        # Socket connection
        client_socket.connect((host, port))
        print(f"Connected to TCP server at {host}:{port}")

        logging.info(f"Connected to TCP server at {host}:{port}")
        logging.info(f"Preparing to send file '{file_name}' of size {file_size} bytes.")
        start_time = time.time()
        logging.info(f"Transmission started at {start_time}.")

        # Send timestamp
        client_socket.sendto(str(start_time).encode(), (host, port))
        time.sleep(0.1)

         # Send file name
        client_socket.sendall(file_name.encode())
        time.sleep(0.1)

        # Send file content
        with open(file_path, 'rb') as f:
            print(f"Sending file '{file_name}'...")
            while chunk := f.read(SEG_SIZE):  # Read and send in chunks
                client_socket.sendall(chunk)

        end_time = time.time()
        logging.info(f"Transmission ended at {end_time}.")
        logging.info(f"Total transmission time: {end_time - start_time - 0.2:.10f} seconds.") # Removes 0.2 because of the sleep
        print(f"File '{file_name}' sent successfully.")

def udp_client(host, port):
    # Ensure the directory exists
    os.makedirs("sent_files", exist_ok=True)
    
    # Ask the user for file details
    file_name = input("Enter the name of the file to use: ").strip()
    file_path = os.path.join("sent_files", file_name)
    file_size = os.path.getsize(file_path)

    with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as client_socket:
        # Disable checksum
        # client_socket.setsockopt(socket.SOL_SOCKET, 11, 1)

        logging.info(f"Preparing to send file '{file_name}' of size {file_size} bytes.")
        start_time = time.time()
        logging.info(f"Transmission started at {start_time}.")
        
        print(f"UDP client ready to send to {host}:{port}")

        # Send timestamp
        client_socket.sendto(str(start_time).encode(), (host, port))
        # Send file name
        client_socket.sendto(file_name.encode(), (host, port))

        # Send file content
        with open(file_path, 'rb') as f:
            print(f"Sending file '{file_name}'...")
            while chunk := f.read(SEG_SIZE):  # Read file in chunks
                client_socket.sendto(chunk, (host, port))

        # Send an EOF marker to indicate the end of the file
        client_socket.sendto(b"EOF", (host, port))

        end_time = time.time()
        logging.info(f"Transmission ended at {end_time}.")
        logging.info(f"Total transmission time: {end_time - start_time:.10f} seconds.")

        print(f"File '{file_name}' sent successfully.")

def create_random_file(file_name, num_bytes):
    """Create a file with random text of the specified size."""
    with open(file_name, 'w') as f:
        while num_bytes > 0:
            chunk_size = min(SEG_SIZE, num_bytes)  # Write in chunks of SEG_SIZE KB or less
            random_text = ''.join(random.choices(string.ascii_letters + string.digits, k=chunk_size))
            f.write(random_text)
            num_bytes -= chunk_size
    print(f"File '{file_name}' created with {os.path.getsize(file_name)} bytes.")


def main():
    # Create files with a specific number of bytes
    # file_name = input("Enter the name of the file to create: ").strip()
    # file_path = os.path.join("sent_files", file_name)
    # num_bytes = int(input("Enter the number of bytes for the file: "))
    # os.makedirs("sent_files", exist_ok=True)
    # create_random_file(file_path, num_bytes)
    
    # List of client types
    client_types = ["TCP", "UDP"]

    # Ask the user to choose a client type
    print("Choose the type of client:")
    for i, client_type in enumerate(client_types, start=1):
        print(f"{i}. {client_type}")

    choice = int(input("Enter your choice (1 or 2): "))
    if choice not in [1, 2]:
        print("Invalid choice. Exiting.")
        return

    # Get host and port from the user
    host = input("Enter the server host (e.g., 127.0.0.1): ").strip()
    port = int(input("Enter the server port (e.g., 12345): "))

    # Run the chosen client
    if choice == 1:
        print("Starting TCP client...")
        tcp_client(host, port)
    elif choice == 2:
        print("Starting UDP client...")
        udp_client(host, port)

if __name__ == "__main__":
    main()
