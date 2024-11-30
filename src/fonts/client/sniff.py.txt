import yaml
import sys
from scapy.all import sniff, IP, TCP
from datetime import datetime

# Substitua pelo IP do ESP32
ESP32_IP = "192.168.0.47"
packets_sniffed_count = 0
packet_list = []


def preprocess_packet_data(packet):
    """Convert packet data to a YAML-friendly format."""
    return {
        "source": f"{packet[IP].src}:{packet[TCP].sport}",
        "destination": f"{packet[IP].dst}:{packet[TCP].dport}",
        "flags": str(packet[TCP].flags),
        "sequence_number": packet[TCP].seq,
        "acknowledgment_number": packet[TCP].ack,
        "checksum": packet[TCP].chksum,
        "urgent_pointer": packet[TCP].urgptr,
        "MSS": packet[TCP].options[0][1] if len(packet[TCP].options) > 0 else None,
        "header_size_words": packet[TCP].dataofs,
        "total_size_bytes": packet[IP].len,
        "window_size": packet[TCP].window,
        "payload_size_bytes": len(packet[TCP].payload),
    }


def write_packet_to_log(packet_list):
    # Cria um arquivo de log com o timestamp atual
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filename = f"log_{timestamp}.yaml"
    # Write to the YAML file
    with open(filename, "a") as file:
        yaml.dump(packet_list, file, default_flow_style=False, sort_keys=False)
        file.write("\n")


def packet_callback(packet):
    global packets_sniffed_count 
    if IP in packet and (packet[IP].dst == ESP32_IP or packet[IP].src == ESP32_IP):
        if TCP in packet:
            packets_sniffed_count += 1
            # Preprocess the packet data
            packet_data = preprocess_packet_data(packet)
            packet_list.append(packet_data)
            # Print the packet data in YAML format
            ##write_packet_to_log(packet_data)

            # Overwrite the current line with the count
            sys.stdout.write(f"\rPacotes capturados: {packets_sniffed_count}")
            sys.stdout.flush()

def print_packet(packet):
    print("=" * 50)
    print(f"Pacote TCP Recebido")
    print(f"  Origem:         {packet[IP].src}:{packet[TCP].sport}")
    print(f"  Destino:        {packet[IP].dst}:{packet[TCP].dport}")
    print(f"  Flags:          {packet[TCP].flags}")
    print(f"  Nº Sequência:   {packet[TCP].seq}")
    print(f"  Nº Acknowledg.: {packet[TCP].ack}")
    print(f"  Checksum:       {packet[TCP].chksum}")
    print(f"  Ponteiro Urg.:  {packet[TCP].urgptr}")
    print(f"  Opções:         {packet[TCP].options}")
    print(f"  Tam. Cabeçalho: {packet[TCP].dataofs} palavras (multiplicar por 4 para bytes)")
    print(f"  Tam. Total:     {packet[IP].len} bytes")
    print(f"  Janela:         {packet[TCP].window}")
    print(f"  Tam. Payload:   {len(packet[TCP].payload)} bytes")
    print("=" * 50)

# Captura os pacotes na interface principal
print("Capturando pacotes...")

sniff(filter=f"host {ESP32_IP}", prn=packet_callback, store=0)

# Salva os pacotes capturados em um arquivo de log
write_packet_to_log(packet_list)

print("\n")