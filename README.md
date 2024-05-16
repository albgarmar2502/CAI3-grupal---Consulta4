# CAI3-grupal---Consulta4
```python

import time
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.asymmetric import padding
from cryptography.hazmat.primitives.asymmetric import rsa

### Generación de claves RSA
def generate_rsa_keys():
    private_key = rsa.generate_private_key(
        public_exponent=65537,
        key_size=2048,
        backend=default_backend()
    )
    public_key = private_key.public_key()
    return private_key, public_key

### Encriptación RSA
def rsa_encrypt(data, public_key):
    ciphertext = public_key.encrypt(
        data.encode(),
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None
        )
    )
    return ciphertext

### Desencriptación RSA
def rsa_decrypt(ciphertext, private_key):
    data = private_key.decrypt(
        ciphertext,
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None
        )
    )
    return data.decode()

### Protocolo de Transferencia 
def oblivious_transfer(client_choice, server_data):
    #### Leer precios desde el archivo
    with open(server_data, 'r') as file:
        precios = file.read().split(',')

    #### Cliente elige un índice 
    client_index = int(client_choice)

    #### Cliente selecciona el precio del vuelo
    selected_price = precios[client_index].strip()

    #### Encriptar el precio seleccionado
    encrypted_price = rsa_encrypt(selected_price, public_key)

    return encrypted_price

### Generar claves RSA
private_key, public_key = generate_rsa_keys()

### Nombre del archivo que contiene los precios de los vuelos
server_data = "precios.txt"

### Cliente elige un índice (0 o 1) para obtener el precio del vuelo
client_choice = input("Ingrese el número del precio según el orden empezando por 0: ")

### Iniciar temporizador
start_time = time.time()

### Ejecutar el protocolo de Transferencia
price = oblivious_transfer(client_choice, server_data)

### Detener temporizador
end_time = time.time()

### Calcular el tiempo transcurrido
elapsed_time = end_time - start_time

### Desencriptar y mostrar el precio del vuelo seleccionado
decrypted_price = rsa_decrypt(price, private_key)
print("El precio del vuelo seleccionado es:", decrypted_price)

### Mostrar el tiempo transcurrido hasta la respuesta
print("Tiempo transcurrido hasta la respuesta:", elapsed_time, "segundos")
