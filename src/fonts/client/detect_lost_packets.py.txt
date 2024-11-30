import yaml

def detect_lost_packets(log_file):
    with open(log_file, "r") as file:
        log_data = yaml.safe_load(file)
    
    # Sort the packets by source and sequence number
    packets = sorted(log_data, key=lambda x: (x['source'], x['sequence_number']))
    
    lost_packets = []
    prev_sequence_number = None
    prev_source = None

    for packet in packets:
        source = packet['source']
        sequence_number = packet['sequence_number']
        
        # Check if we're still processing the same source
        if source == prev_source:
            # Detect if there's a gap in the sequence numbers
            if prev_sequence_number is not None and sequence_number != prev_sequence_number + 1:
                lost_packet_range = (prev_sequence_number + 1, sequence_number - 1)
                lost_packets.append({
                    "source": source,
                    "lost_packet_range": lost_packet_range
                })
        
        prev_sequence_number = sequence_number
        prev_source = source
    
    return lost_packets

# Example usage
log_file = "log_20241126_103511.yaml"  # Replace with your actual file name
lost_packets = detect_lost_packets(log_file)

# Print the lost packets details
if lost_packets:
    for lost_packet in lost_packets:
        print(f"Source: {lost_packet['source']} | Lost Packet Range: {lost_packet['lost_packet_range']}")
else:
    print("No lost packets detected.")
