import serial
import time
from datetime import datetime

# Configuration
PORT = "/dev/ttyUSB0"  # Serial port
BAUD_RATE = 115200  # Match the ESP32's baud rate
LOG_FILE = "esp32_log_udp_2.txt"  # Log file name

def main():
    try:
        # Open the serial connection
        with serial.Serial(PORT, BAUD_RATE, timeout=1) as ser, open(LOG_FILE, 'a') as log_file:
            print(f"Listening on {PORT} at {BAUD_RATE} baud...")
            while True:
                # Read a line from the ESP32
                if ser.in_waiting > 0:
                    message = ser.readline().decode('utf-8').strip()
                    if message:
                        # Add a timestamp
                        timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
                        #log_entry = f"[{timestamp}] {message}\n"
                        log_entry = f"{message}\n"
                        
                        # Print to console and save to log
                        print(log_entry, end='')
                        log_file.write(log_entry)
                        log_file.flush()  # Ensure data is written immediately
    except serial.SerialException as e:
        print(f"Serial error: {e}")
    except KeyboardInterrupt:
        print("\nExiting...")

if __name__ == "__main__":
    main()
