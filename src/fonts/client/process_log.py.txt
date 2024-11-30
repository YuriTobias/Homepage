import yaml

# Load the YAML file
with open("log_20241126_103511.yaml", "r") as file:
    data = yaml.safe_load(file)

# Now `data` is a Python dictionary, and you can analyze it.
#print(data)  # Prints the loaded YAML data

for packet in data:
    if packet["source"] == "192.168.0.47:8080":
        print(f"Tamanho da Janela: {packet['window_size']} bytes")
