# Simulador-de-Ransomware
#Un pequeño codigo intituivo de como funciona un ransomware basado en python usando tkinter en debian

import tkinter as tk
from PIL import ImageTk, Image
import pyotp
import qrcode
import os

# --- 1. CONFIGURACIÓN TÉCNICA (2FA) ---
# Clave Base32 para Google Authenticator
key = "GEEKSFORGEEKSISBESTFOREVERYTHING" 
totp = pyotp.TOTP(key)

# --- 2. DICCIONARIO DE TRADUCCIONES ---
traducciones = {
    "es": {
        "titulo": "⚠ ARCHIVOS BLOQUEADOS ⚠",
        "instruccion": "Escanea el QR y pon el código:",
        "boton": "DESBLOQUEAR",
        "reloj": "BORRADO EN: ",
        "pagar": "PAGAR RESCATE (1 BTC)",
        "ganar": "¡ACCESO CONCEDIDO!\nArchivos desencriptados.",
        "perder": "SISTEMA DESTRUIDO\nHas perdido tus datos.",
        "estafa": "¡ESTAFA DETECTADA!\nLos hackers han borrado todo.",
        "menu": "Idioma"
    },
    "cat": {
        "titulo": "⚠ ARXIUS BLOQUEJATS ⚠",
        "instruccion": "Escaneja el QR i posa el codi:",
        "boton": "DESBLOQUEJAR",
        "reloj": "ESBORRAT EN: ",
        "pagar": "PAGAR RESCAT (1 BTC)",
        "ganar": "¡ACCÉS CONCEDIT!\nArxius desencriptats.",
        "perder": "SISTEMA DESTRUÏT\nHas perdut les dades.",
        "estafa": "¡ESTAFA DETECTADA!\nEls hackers han esborrat tot.",
        "menu": "Llengua"
    },
    "en": {
        "titulo": "⚠ FILES ENCRYPTED ⚠",
        "instruccion": "Scan QR and enter code:",
        "boton": "UNLOCK",
        "reloj": "DELETION IN: ",
        "pagar": "PAY RANSOM (1 BTC)",
        "ganar": "ACCESS GRANTED!\nFiles decrypted.",
        "perder": "SYSTEM DESTROYED\nYou lost your data.",
        "estafa": "SCAM DETECTED!\nHackers deleted everything.",
        "menu": "Language"
    }
}

idioma_actual = "es"
tiempo_restante = 10

# --- 3. FUNCIONES DE LÓGICA ---

def preparar_qr():
    """Genera el QR inicial"""
    uri = totp.provisioning_uri(name='Debian_User', issuer_name='RansomSim')
    img_qr = qrcode.make(uri)
    img_qr.save("temp_qr.png")

def cambiar_idioma(nuevo_idioma):
    global idioma_actual
    idioma_actual = nuevo_idioma
    lbl_titulo.config(text=traducciones[idioma_actual]["titulo"])
    lbl_instruccion.config(text=traducciones[idioma_actual]["instruccion"])
    btn_check.config(text=traducciones[idioma_actual]["boton"])
    btn_pagar.config(text=traducciones[idioma_actual]["pagar"])
    barra_menus.entryconfig(1, label=traducciones[idioma_actual]["menu"])

def verificar_codigo():
    if totp.verify(entrada_2fa.get()):
        finalizar_juego(traducciones[idioma_actual]["ganar"], "green")
    else:
        lbl_status.config(text="X", fg="yellow") # Error rápido
        entrada_2fa.delete(0, tk.END)

def pagar_rescate():
    """Botón trampa: El usuario pierde por confiar en el hacker"""
    finalizar_juego(traducciones[idioma_actual]["estafa"], "darkred")

def actualizar_reloj():
    global tiempo_restante
    if tiempo_restante > 0:
        tiempo_restante -= 1
        lbl_reloj.config(text=f"{traducciones[idioma_actual]['reloj']}{tiempo_restante}s")
        
        # SONIDO PARA DEBIAN (requiere sox)
        # El pitido sube de tono (Hz) a medida que queda menos tiempo
        frecuencia = 800 + (60 - tiempo_restante) * 10
        os.system(f"play -nq -t alsa synth 0.1 sine {frecuencia} > /dev/null 2>&1 &")
        
        # Efecto visual de parpadeo
        color = "red" if tiempo_restante % 2 == 0 else "darkred"
        ventana.config(bg=color)
        lbl_instruccion.config(bg=color)
        lbl_status.config(bg=color)
        ventana.after(1000, actualizar_reloj)
    else:
        finalizar_juego(traducciones[idioma_actual]["perder"], "black")

def finalizar_juego(msg, color):
    for widget in ventana.winfo_children():
        widget.destroy()
    ventana.config(bg=color)
    tk.Label(ventana, text=msg, fg="white", bg=color, font=("Courier", 16, "bold"), justify="center").pack(expand=True)
    # Limpieza del archivo QR
    if os.path.exists("temp_qr.png"):
        os.remove("temp_qr.png")

# --- 4. INTERFAZ GRÁFICA ---

preparar_qr()
ventana = tk.Tk()
ventana.title("Ransomware Simulator - Security Project")
ventana.geometry("500x750")
ventana.config(bg="darkred")

# Menú superior
barra_menus = tk.Menu(ventana)
menu_idioma = tk.Menu(barra_menus, tearoff=0)
menu_idioma.add_command(label="Castellano", command=lambda: cambiar_idioma("es"))
menu_idioma.add_command(label="Català", command=lambda: cambiar_idioma("cat"))
menu_idioma.add_command(label="English", command=lambda: cambiar_idioma("en"))
barra_menus.add_cascade(label="Idioma", menu=menu_idioma)
ventana.config(menu=barra_menus)

# Widgets principales
lbl_titulo = tk.Label(ventana, text=traducciones[idioma_actual]["titulo"], 
                      fg="white", bg="black", font=("Courier", 20, "bold"))
lbl_titulo.pack(fill="x")

# Carga de imagen
img = Image.open("temp_qr.png")
img = img.resize((240, 240))
photo = ImageTk.PhotoImage(img)
lbl_qr = tk.Label(ventana, image=photo, borderwidth=5, relief="ridge")
lbl_qr.pack(pady=20)

lbl_instruccion = tk.Label(ventana, text=traducciones[idioma_actual]["instruccion"], 
                           fg="white", bg="darkred", font=("Courier", 10))
lbl_instruccion.pack()

lbl_reloj = tk.Label(ventana, text="", fg="white", bg="black", font=("Courier", 14, "bold"))
lbl_reloj.pack(pady=15, fill="x")

entrada_2fa = tk.Entry(ventana, font=("Arial", 28), justify="center", width=8)
entrada_2fa.pack(pady=5)
entrada_2fa.focus_set()

btn_check = tk.Button(ventana, text=traducciones[idioma_actual]["boton"], 
                      command=verificar_codigo, bg="#00FF00", fg="black", 
                      font=("Courier", 12, "bold"), height=2, width=22)
btn_check.pack(pady=15)

# BOTÓN TRAMPA
btn_pagar = tk.Button(ventana, text=traducciones[idioma_actual]["pagar"], 
                      command=pagar_rescate, bg="black", fg="yellow", 
                      font=("Courier", 10, "italic"), relief="flat")
btn_pagar.pack(pady=5)

lbl_status = tk.Label(ventana, text="", bg="darkred", fg="white", font=("Courier", 12))
lbl_status.pack()

# Iniciar procesos
actualizar_reloj()
ventana.mainloop()
