# Implementando envio de notificação

---

## **Passo 1: Configurar o Sensor no Packet Tracer**
1. **Criar o Ambiente no Packet Tracer**:
   - Adicione um **sensor de movimento** ao ambiente.
   - Adicione um **dispositivo controlador** (como um servidor ou um dispositivo IoT compatível).

2. **Configurar o Sensor**:
   - Acesse as configurações do sensor.
   - Configure o comportamento do sensor para enviar uma requisição HTTP (por exemplo, um POST) quando detectar movimento.
   - Informe o endereço IP e a porta do servidor Python que será executado posteriormente.

3. **Simular a Rede**:
   - Adicione um **roteador** e conecte os dispositivos (sensor e servidor) ao roteador.
   - Certifique-se de configurar corretamente os endereços IP dos dispositivos.

4. **Testar o Sensor**:
   - Execute a simulação no Packet Tracer.
   - Garanta que o sensor envia uma requisição HTTP quando detecta movimento (você pode usar ferramentas simples no servidor, como `Packet Sniffer`, para verificar).

---

## **Passo 2: Criar o Servidor Python (HTTP + E-mail)**
1. **Configurar o Servidor HTTP com Flask**:
   - Instale o Flask:
     ```bash
     pip install flask
     ```
   - Crie um script Python para o servidor HTTP. Exemplo:
     ```python
     from flask import Flask, request
     import smtplib

     app = Flask(__name__)

     @app.route('/sensor-alert', methods=['POST'])
     def sensor_alert():
         try:
             # Enviar e-mail ao receber notificação do sensor
             server = smtplib.SMTP('smtp.gmail.com', 587)
             server.starttls()
             server.login('seu_email@gmail.com', 'sua_senha')

             mensagem = 'Subject: Alerta do Sensor\n\nMovimento detectado pelo sensor.'
             server.sendmail('seu_email@gmail.com', 'destinatario@gmail.com', mensagem)
             server.quit()

             return "E-mail enviado com sucesso!", 200
         except Exception as e:
             return f"Erro: {str(e)}", 500

     if __name__ == '__main__':
         app.run(host='0.0.0.0', port=5000)
     ```
   - Substitua `seu_email@gmail.com`, `sua_senha` e `destinatario@gmail.com` pelas suas credenciais e destinatário.

2. **Testar o Servidor**:
   - Execute o servidor Python e envie uma requisição HTTP manualmente (use uma ferramenta como **Postman** ou `curl`):
     ```bash
     curl -X POST http://localhost:5000/sensor-alert
     ```

3. **Conectar ao Packet Tracer**:
   - Ajuste o endereço IP no Packet Tracer para apontar para o servidor Flask (o IP do seu computador na rede simulada).

---

## **Passo 3: Implementar o Script para Ler E-mails**
1. **Configurar a Biblioteca IMAP**:
   - Certifique-se de que o IMAP está ativado no seu provedor de e-mail (exemplo: em contas Gmail, vá em Configurações > Encaminhamento e POP/IMAP).

2. **Criar o Script Python para Leitura de E-mails**:
   - Exemplo:
     ```python
     import imaplib
     import email
     from email.header import decode_header

     # Configurações de e-mail
     IMAP_SERVER = 'imap.gmail.com'
     EMAIL = 'seu_email@gmail.com'
     PASSWORD = 'sua_senha'

     def verificar_emails():
         mail = imaplib.IMAP4_SSL(IMAP_SERVER)
         mail.login(EMAIL, PASSWORD)
         mail.select("inbox")
         
         status, mensagens = mail.search(None, 'UNSEEN')
         emails_ids = mensagens[0].split()

         for e_id in emails_ids:
             status, msg_data = mail.fetch(e_id, '(RFC822)')
             for response_part in msg_data:
                 if isinstance(response_part, tuple):
                     msg = email.message_from_bytes(response_part[1])
                     assunto, encoding = decode_header(msg["Subject"])[0]
                     if isinstance(assunto, bytes):
                         assunto = assunto.decode(encoding if encoding else "utf-8")

                     if "Movimento detectado" in assunto:
                         print("Alerta recebido!")
                         return True
         mail.logout()
         return False
     ```

3. **Testar o Script**:
   - Execute o script e envie um e-mail de teste com o assunto "Movimento detectado" para verificar se ele é identificado.

---

## **Passo 4: Criar o App Tkinter**
1. **Criar a Interface Gráfica**:
   - Use Tkinter para criar um aplicativo básico que muda de cor ao detectar o e-mail.
   - Exemplo:
     ```python
     import tkinter as tk
     import time
     from threading import Thread
     from email_script import verificar_emails  # Importa sua função de leitura de e-mails

     def monitorar_emails():
         while True:
             if verificar_emails():
                 alterar_cor("red")
             time.sleep(10)

     def alterar_cor(cor):
         tela.config(bg=cor)

     tela = tk.Tk()
     tela.title("Monitor de Sensor")
     tela.geometry("400x300")
     tela.config(bg="white")

     rotulo = tk.Label(tela, text="Monitorando e-mails...", font=("Arial", 16))
     rotulo.pack(pady=20)

     Thread(target=monitorar_emails, daemon=True).start()
     tela.mainloop()
     ```

2. **Integrar com o Script de Leitura**:
   - Certifique-se de que o leitor de e-mails está funcionando e chamando a função `alterar_cor` ao detectar o e-mail.

---

## **Passo 5: Testar o Sistema**
1. **Executar o Servidor Flask**:
   - Certifique-se de que o servidor Flask está em execução e conectado ao Packet Tracer.

2. **Testar o Sensor no Packet Tracer**:
   - Simule um disparo do sensor para garantir que ele envia a mensagem HTTP corretamente.

3. **Monitorar o E-mail e o Tkinter**:
   - Verifique se o e-mail é recebido e, em seguida, se o app Tkinter altera o fundo conforme esperado.

---

### Resultado Final:
- Quando o sensor no Packet Tracer detectar movimento:
  1. Ele enviará uma requisição HTTP para o servidor Python.
  2. O servidor enviará um e-mail para sua conta.
  3. O script de leitura de e-mails detectará o e-mail.
  4. O app Tkinter mudará a cor da interface.
