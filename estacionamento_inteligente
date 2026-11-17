import tkinter as tk
import serial
import time
import random
from collections import deque

# =====================================
# CONFIGURACOES
# =====================================

PORTA = "COM3"
BAUDRATE = 9600
TOTAL_VAGAS = 2

# =====================================
# CONECTAR AO ARDUINO
# =====================================

try:
    arduino = serial.Serial(
        port=PORTA,
        baudrate=BAUDRATE,
        timeout=0.1
    )

    time.sleep(2)
    arduino.reset_input_buffer()
    print("ARDUINO CONECTADO COM SUCESSO!")

except serial.SerialException as erro:
    print("ERRO AO CONECTAR:")
    print(erro)
    raise SystemExit

# =====================================
# VARIAVEIS
# =====================================

vagas = [False] * TOTAL_VAGAS
ordem_saida = deque()
ultimo_evento = ""

# =====================================
# JANELA PRINCIPAL
# =====================================

janela = tk.Tk()
janela.title("Estacionamento Inteligente")
janela.configure(bg="#1e1e2f")
janela.minsize(800, 600)

# Maximizar automaticamente no Windows
try:
    janela.state("zoomed")
except tk.TclError:
    janela.attributes("-fullscreen", True)

# =====================================
# CORES
# =====================================

FUNDO = "#1e1e2f"
CARD = "#2d3748"
VERDE = "#00F148"
VERMELHO = "#e74c3c"
AZUL = "#60a5fa"
BRANCO = "#ffffff"

# =====================================
# ENVIAR COMANDO AO ARDUINO
# =====================================

def enviar_comando(comando):
    try:
        if arduino.is_open:
            arduino.write((comando + "\n").encode("utf-8"))
    except serial.SerialException:
        atualizar_conexao(False)


# =====================================
# STATUS DA CONEXAO
# =====================================

def atualizar_conexao(estado):
    if estado:
        status_conexao.config(
            text="● ARDUINO CONECTADO",
            fg=VERDE
        )
    else:
        status_conexao.config(
            text="● ARDUINO DESCONECTADO",
            fg=VERMELHO
        )


# =====================================
# ATUALIZAR CANCELA E LEDS
# =====================================

def atualizar_hardware():
    livres = vagas.count(False)

    enviar_comando(
        f"VAGAS,{int(vagas[0])},{int(vagas[1])}"
    )

    if livres == 0:
        enviar_comando("CANCELA,FECHAR")

        status_cancela.config(
            text="FECHADA",
            fg=VERMELHO
        )

    else:
        enviar_comando("CANCELA,ABRIR")

        status_cancela.config(
            text="ABERTA",
            fg=VERDE
        )



# =====================================
# ATUALIZAR TELA
# =====================================

def atualizar_tela():
    livres = vagas.count(False)
    ocupadas = vagas.count(True)

    atualizar_vaga_disponivel.config(text=str(livres))
    atualizar_vaga_ocupada.config(text=str(ocupadas))

    for i in range(TOTAL_VAGAS):
        if vagas[i]:
            status_vagas[i].config(
                text="OCUPADA",
                fg=VERMELHO
            )
        else:
            status_vagas[i].config(
                text="LIVRE",
                fg=VERDE
            )

    atualizar_hardware()


# =====================================
# ENTRADA DE VEICULO
# =====================================

def entrada_carro():
    global ultimo_evento

    vagas_livres = [
        i for i in range(TOTAL_VAGAS)
        if not vagas[i]
    ]

    if not vagas_livres:
        status_evento.config(
            text="ESTACIONAMENTO LOTADO!",
            fg=VERMELHO
        )
        atualizar_tela()
        return

    vaga = random.choice(vagas_livres)

    vagas[vaga] = True
    ordem_saida.append(vaga)

    ultimo_evento = "ENTRADA"

    status_evento.config(
        text=f"CARRO ENTROU - VAGA {vaga + 1} OCUPADA",
        fg=VERDE
    )

    atualizar_tela()


# =====================================
# SAIDA DE VEICULO
# =====================================

def saida_carro():
    global ultimo_evento

    if not ordem_saida:
        status_evento.config(
            text="NENHUM CARRO REGISTRADO!",
            fg=VERMELHO
        )
        return

    vaga = ordem_saida.popleft()
    vagas[vaga] = False

    ultimo_evento = "SAIDA"

    status_evento.config(
        text=f"CARRO SAIU - VAGA {vaga + 1} LIBERADA",
        fg=AZUL
    )

    atualizar_tela()


# =====================================
# RECEBER MENSAGENS DO ARDUINO
# =====================================

def verificar_serial():
    try:
        while arduino.in_waiting > 0:
            mensagem = arduino.readline().decode(
                "utf-8",
                errors="ignore"
            ).strip().upper()

            if not mensagem:
                continue

            # ENTRADA
            if (
                mensagem == "ENTRADA"
                or "CARRO DETECTADO NA ENTRADA" in mensagem
            ):
                entrada_carro()

            # SAIDA
            elif (
                mensagem == "SAIDA"
                or "CARRO DETECTADO NA SAIDA" in mensagem
            ):
                saida_carro()

    except serial.SerialException:
        atualizar_conexao(False)

    if janela.winfo_exists():
        janela.after(50, verificar_serial)


# =====================================
# FECHAR PROGRAMA
# =====================================

def fechar_programa():
    try:
        if arduino.is_open:
            arduino.close()
    except serial.SerialException:
        pass

    janela.destroy()


# =====================================
# CABECALHO
# =====================================

cabecalho = tk.Frame(janela, bg=FUNDO)
cabecalho.pack(fill="x", pady=(10, 2))

tk.Label(
    cabecalho,
    text="ESTACIONAMENTO INTELIGENTE",
    font=("Arial", 26, "bold"),
    fg=BRANCO,
    bg=FUNDO
).pack()

status_conexao = tk.Label(
    cabecalho,
    text="● ARDUINO CONECTADO",
    font=("Arial", 12, "bold"),
    fg=VERDE,
    bg=FUNDO
)
status_conexao.pack(pady=3)


# =====================================
# STATUS DO EVENTO
# =====================================

status_evento = tk.Label(
    janela,
    text="AGUARDANDO VEICULO...",
    font=("Arial", 14, "bold"),
    fg=AZUL,
    bg=FUNDO
)
status_evento.pack(pady=5)


# =====================================
# CONTEUDO PRINCIPAL
# =====================================

conteudo = tk.Frame(janela, bg=FUNDO)
conteudo.pack(
    fill="both",
    expand=True,
    padx=25,
    pady=5
)


# =====================================
# VAGAS DISPONIVEIS
# =====================================

card_disponiveis = tk.Frame(
    conteudo,
    bg=CARD,
    height=175
)
card_disponiveis.pack(
    fill="x",
    pady=5
)
card_disponiveis.pack_propagate(False)

tk.Label(
    card_disponiveis,
    text="VAGAS DISPONIVEIS",
    font=("Arial", 18, "bold"),
    fg=BRANCO,
    bg=CARD
).pack(pady=(12, 0))

atualizar_vaga_disponivel = tk.Label(
    card_disponiveis,
    text="2",
    font=("Arial", 65, "bold"),
    fg=VERDE,
    bg=CARD
)
atualizar_vaga_disponivel.pack(pady=0)


# =====================================
# CARDS INFERIORES
# =====================================

frame_inferior = tk.Frame(conteudo, bg=FUNDO)
frame_inferior.pack(
    fill="x",
    pady=5
)

frame_inferior.grid_columnconfigure(0, weight=1)
frame_inferior.grid_columnconfigure(1, weight=1)

card_capacidade = tk.Frame(
    frame_inferior,
    bg=CARD,
    height=125
)
card_capacidade.grid(
    row=0,
    column=0,
    padx=(0, 5),
    sticky="nsew"
)
card_capacidade.pack_propagate(False)

tk.Label(
    card_capacidade,
    text="CAPACIDADE MAXIMA",
    font=("Arial", 15, "bold"),
    fg=BRANCO,
    bg=CARD
).pack(pady=(10, 0))

tk.Label(
    card_capacidade,
    text=str(TOTAL_VAGAS),
    font=("Arial", 43, "bold"),
    fg=BRANCO,
    bg=CARD
).pack(pady=0)


card_ocupadas = tk.Frame(
    frame_inferior,
    bg=CARD,
    height=125
)
card_ocupadas.grid(
    row=0,
    column=1,
    padx=(5, 0),
    sticky="nsew"
)
card_ocupadas.pack_propagate(False)

tk.Label(
    card_ocupadas,
    text="VAGAS OCUPADAS",
    font=("Arial", 15, "bold"),
    fg=BRANCO,
    bg=CARD
).pack(pady=(10, 0))

atualizar_vaga_ocupada = tk.Label(
    card_ocupadas,
    text="0",
    font=("Arial", 43, "bold"),
    fg=VERMELHO,
    bg=CARD
)
atualizar_vaga_ocupada.pack(pady=0)


# =====================================
# STATUS DAS VAGAS
# =====================================

tk.Label(
    conteudo,
    text="STATUS DAS VAGAS",
    font=("Arial", 17, "bold"),
    fg=BRANCO,
    bg=FUNDO
).pack(pady=(5, 7))

frame_vagas = tk.Frame(conteudo, bg=FUNDO)
frame_vagas.pack(fill="x")

status_vagas = []

for i in range(TOTAL_VAGAS):
    card = tk.Frame(
        frame_vagas,
        bg=CARD,
        height=90
    )

    card.pack(
        side="left",
        fill="both",
        expand=True,
        padx=5
    )

    card.pack_propagate(False)

    tk.Label(
        card,
        text=f"VAGA {i + 1}",
        font=("Arial", 15, "bold"),
        fg=BRANCO,
        bg=CARD
    ).pack(pady=7)

    label = tk.Label(
        card,
        text="LIVRE",
        font=("Arial", 19, "bold"),
        fg=VERDE,
        bg=CARD
    )
    label.pack()

    status_vagas.append(label)


# =====================================
# STATUS DA CANCELA
# =====================================

frame_cancela = tk.Frame(
    conteudo,
    bg=CARD,
    height=85
)
frame_cancela.pack(
    fill="x",
    pady=8
)
frame_cancela.pack_propagate(False)

tk.Label(
    frame_cancela,
    text="STATUS DA CANCELA",
    font=("Arial", 14, "bold"),
    fg=BRANCO,
    bg=CARD
).pack(pady=(5, 0))

status_cancela = tk.Label(
    frame_cancela,
    text="ABERTA",
    font=("Arial", 21, "bold"),
    fg=VERDE,
    bg=CARD
)
status_cancela.pack(pady=0)


# =====================================
# INICIALIZAR
# =====================================

atualizar_tela()

janela.after(100, verificar_serial)

janela.protocol(
    "WM_DELETE_WINDOW",
    fechar_programa
)

janela.mainloop()
