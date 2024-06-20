import socket
from threading import Thread


# Define server address and port
server_addr = ('0.0.0.0', 9999)

# Create a UDP socket
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.bind(server_addr)

# Dictionary to store users and their addresses
clientsNames = {}
clientsAddr = {}

def handle_message(data, addr):
    message = data.decode()
    parts = message.split(' ', 1)
    name = parts[0]
    if addr not in clientsNames.values():
        # Authenticate sender
        clientsNames[name] = addr
        clientsAddr[addr] = name
        print(f"{name} has joined and the addr is {addr}.")
    else:
        # Forward message
        if name in clientsNames.keys():
            msg= parts[1]
            print(msg,",",clientsNames[name])
            try:
                sock.sendto(f"from {clientsAddr[addr]}: {msg}".encode(),clientsNames[name])
            except socket.error as e:
                print(f"Error receiving data: {e}")
        else:
            try:
                sock.sendto(f"{name} not such user".encode(),addr)
            except socket.error as e:
                print(f"Error receiving data: {e}")

while True:
    # Receive message from client
    data, addr = sock.recvfrom(1024)
    Thread(target=handle_message, args=(data,addr)).start()
    