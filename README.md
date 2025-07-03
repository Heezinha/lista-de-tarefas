# lista-de-tarefas
import tkinter as tk

# ConfiguraÃ§Ãµes
LARGURA = 1000
ALTURA = 800
TAMANHO = 10
VELOCIDADE = 50  # milissegundos por movimento

class Tron:
    def __init__(self, master):
        self.master = master
        self.master.title("Tron - 2 Jogadores")
        self.canvas = tk.Canvas(master, width=LARGURA, height=ALTURA, bg="black")
        self.canvas.pack()

        self.resetar_jogo()

        self.master.bind("<KeyPress>", self.tecla_pressionada)
        self.atualizar()

    def resetar_jogo(self):
        self.jogo_ativo = True

        # Jogador 1 (vermelho)
        self.j1_pos = [100, 200]
        self.j1_direcao = "Right"
        self.j1_trilha = [tuple(self.j1_pos)]
        self.j1_cor = "red"

        # Jogador 2 (azul)
        self.j2_pos = [500, 200]
        self.j2_direcao = "Left"
        self.j2_trilha = [tuple(self.j2_pos)]
        self.j2_cor = "blue"

        self.canvas.delete("all")

    def tecla_pressionada(self, event):
        tecla = event.keysym

        # Controles jogador 1 (WASD)
        if tecla == "w" and self.j1_direcao != "Down":
            self.j1_direcao = "Up"
        elif tecla == "s" and self.j1_direcao != "Up":
            self.j1_direcao = "Down"
        elif tecla == "a" and self.j1_direcao != "Right":
            self.j1_direcao = "Left"
        elif tecla == "d" and self.j1_direcao != "Left":
            self.j1_direcao = "Right"

        # Controles jogador 2 (setas)
        elif tecla == "Up" and self.j2_direcao != "Down":
            self.j2_direcao = "Up"
        elif tecla == "Down" and self.j2_direcao != "Up":
            self.j2_direcao = "Down"
        elif tecla == "Left" and self.j2_direcao != "Right":
            self.j2_direcao = "Left"
        elif tecla == "Right" and self.j2_direcao != "Left":
            self.j2_direcao = "Right"

    def mover(self, pos, direcao):
        x, y = pos
        if direcao == "Up":
            y -= TAMANHO
        elif direcao == "Down":
            y += TAMANHO
        elif direcao == "Left":
            x -= TAMANHO
        elif direcao == "Right":
            x += TAMANHO
        return [x, y]

    def atualizar(self):
        if not self.jogo_ativo:
            return

        # Mover jogadores
        self.j1_pos = self.mover(self.j1_pos, self.j1_direcao)
        self.j2_pos = self.mover(self.j2_pos, self.j2_direcao)

        # Verificar colisÃµes
        todos_rastros = set(self.j1_trilha + self.j2_trilha)
        col_j1 = tuple(self.j1_pos) in todos_rastros or not self.dentro_limite(self.j1_pos)
        col_j2 = tuple(self.j2_pos) in todos_rastros or not self.dentro_limite(self.j2_pos)

        # Atualizar trilhas
        self.j1_trilha.append(tuple(self.j1_pos))
        self.j2_trilha.append(tuple(self.j2_pos))

        # Desenhar trilhas
        self.canvas.create_rectangle(self.j1_pos[0], self.j1_pos[1],
                                     self.j1_pos[0]+TAMANHO, self.j1_pos[1]+TAMANHO,
                                     fill=self.j1_cor, outline=self.j1_cor)

        self.canvas.create_rectangle(self.j2_pos[0], self.j2_pos[1],
                                     self.j2_pos[0]+TAMANHO, self.j2_pos[1]+TAMANHO,
                                     fill=self.j2_cor, outline=self.j2_cor)

        if col_j1 and col_j2:
            self.fim_de_jogo("Empate!")
        elif col_j1:
            self.fim_de_jogo("Jogador 2 venceu!")
        elif col_j2:
            self.fim_de_jogo("Jogador 1 venceu!")
        else:
            self.master.after(VELOCIDADE, self.atualizar)

    def dentro_limite(self, pos):
        x, y = pos
        return 0 <= x < LARGURA and 0 <= y < ALTURA

    def fim_de_jogo(self, mensagem):
        self.jogo_ativo = False
        self.canvas.create_text(LARGURA//2, ALTURA//2, fill="white",
                                font=("Arial", 20, "bold"), text=mensagem)

        self.canvas.create_text(LARGURA//2, ALTURA//2 + 30, fill="white",
                                font=("Arial", 12), text="Reiniciando em 3 segundos...")
        self.master.after(3000, self.resetar_jogo)
        self.master.after(3000, self.atualizar)

# Iniciar jogo
if __name__ == "__main__":
    root = tk.Tk()
    game = Tron(root)
    root.mainloop()
