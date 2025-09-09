#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define MAX_CARTAS 10
#define NUM_ATRIBUTOS 4

// Estrutura de uma carta
typedef struct {
    char nome[40];
    int atributos[NUM_ATRIBUTOS]; // 0: Velocidade, 1: Potência, 2: Peso, 3: Autonomia
} Carta;

// Estrutura de um jogador
typedef struct {
    char nome[30];
    Carta cartas[MAX_CARTAS];
    int total;
} Jogador;

// Lista de categorias
const char* categorias[NUM_ATRIBUTOS] = {"Velocidade", "Potência", "Peso", "Autonomia"};

// Lista de nomes de carros
const char* nomes_carros[MAX_CARTAS] = {
    "Ferrari 488 GTB",
    "Lamborghini Huracan",
    "Porsche 911 Turbo S",
    "McLaren 720S",
    "Audi R8 V10 Plus",
    "Nissan GT-R Nismo",
    "Ford GT",
    "Chevrolet Corvette Z06",
    "Mercedes-AMG GT R",
    "Aston Martin Vantage"
};

// --- Funções do Jogo ---

// Funcao para limpar o buffer do teclado
void limparBuffer() {
    int c;
    while ((c = getchar()) != '\n' && c != EOF);
}

// Função para embaralhar o baralho
void embaralhar(Carta *baralho, int n) {
    for (int i = 0; i < n; i++) {
        int j = rand() % n;
        Carta temp = baralho[i];
        baralho[i] = baralho[j];
        baralho[j] = temp;
    }
}

// Função para criar o baralho inicial
void criarBaralho(Carta *baralho, int n) {
    for (int i = 0; i < n; i++) {
        strcpy(baralho[i].nome, nomes_carros[i]);
        baralho[i].atributos[0] = rand() % 300 + 100;    // Velocidade (100-400)
        baralho[i].atributos[1] = rand() % 500 + 50;     // Potência (50-550)
        baralho[i].atributos[2] = rand() % 2000 + 800;   // Peso (800-2800)
        baralho[i].atributos[3] = rand() % 1000 + 200;   // Autonomia (200-1200)
    }
}

// Função para mostrar os detalhes de uma carta
void mostrarCarta(Carta c) {
    printf("--- %s ---\n", c.nome);
    for (int i = 0; i < NUM_ATRIBUTOS; i++) {
        printf("%s: %d\n", categorias[i], c.atributos[i]);
    }
    printf("-------------------\n");
}

// Função para gerenciar o movimento de cartas
void moverCarta(Jogador *vencedor, Jogador *perdedor) {
    // Adiciona a carta do perdedor ao fundo do baralho do vencedor
    vencedor->cartas[vencedor->total++] = perdedor->cartas[0];
    
    // Move todas as cartas do perdedor uma posição para cima
    for (int i = 0; i < perdedor->total - 1; i++) {
        perdedor->cartas[i] = perdedor->cartas[i+1];
    }
    perdedor->total--;
}

// Função para lidar com o empate
void gerenciarEmpate(Jogador *jogador1, Jogador *jogador2, Carta *pote, int *tamanho_pote) {
    // Move as cartas para o pote
    pote[(*tamanho_pote)++] = jogador1->cartas[0];
    pote[(*tamanho_pote)++] = jogador2->cartas[0];

    // Remove as cartas dos jogadores
    for (int i = 0; i < jogador1->total - 1; i++) {
        jogador1->cartas[i] = jogador1->cartas[i+1];
    }
    jogador1->total--;

    for (int i = 0; i < jogador2->total - 1; i++) {
        jogador2->cartas[i] = jogador2->cartas[i+1];
    }
    jogador2->total--;

    printf("\nEmpate! As cartas foram para o pote. A proxima rodada vale mais!\n");
}


// Função principal do jogo
int main() {
    srand(time(NULL));

    Carta baralho[MAX_CARTAS];
    criarBaralho(baralho, MAX_CARTAS);
    embaralhar(baralho, MAX_CARTAS);

    Jogador voce = {"Você", {0}, 0};
    Jogador cpu = {"Computador", {0}, 0};

    // Pote para cartas em caso de empate
    Carta pote[MAX_CARTAS];
    int tamanho_pote = 0;
    
    // Variável para controlar de quem é o turno
    int turno_jogador = 1; // 1 para Você, 0 para CPU

    // Dividindo as cartas
    for (int i = 0; i < MAX_CARTAS; i++) {
        if (i % 2 == 0) {
            voce.cartas[voce.total++] = baralho[i];
        } else {
            cpu.cartas[cpu.total++] = baralho[i];
        }
    }

    printf("Bem-vindo ao Super Trunfo!\n");
    printf("As cartas foram distribuidas. Vamos comecar o jogo.\n\n");

    // Loop principal do jogo
    while (voce.total > 0 && cpu.total > 0) {
        printf("\n--- Rodada ---\n");
        printf("Você tem %d cartas. O computador tem %d cartas.\n\n", voce.total, cpu.total);

        int escolha;
        int vencedor_rodada = -1; // -1: empate, 0: CPU, 1: Você

        // Exibe a carta do jogador
        printf("Sua carta atual:\n");
        mostrarCarta(voce.cartas[0]);

        if (turno_jogador == 1) {
            // Turno do jogador humano
            do {
                printf("Escolha um atributo (1-%d):\n", NUM_ATRIBUTOS);
                for (int i = 0; i < NUM_ATRIBUTOS; i++) {
                    printf("%d. %s\n", i + 1, categorias[i]);
                }
                printf("Sua escolha: ");
                scanf("%d", &escolha);
                limparBuffer();
                escolha--; // Ajusta para o índice do array (0-3)

                if (escolha < 0 || escolha >= NUM_ATRIBUTOS) {
                    printf("Escolha invalida. Tente novamente.\n");
                }
            } while (escolha < 0 || escolha >= NUM_ATRIBUTOS);

            printf("\nVocê escolheu: %s\n", categorias[escolha]);
        } else {
            // Turno do computador (escolha aleatória)
            escolha = rand() % NUM_ATRIBUTOS;
            printf("O computador escolheu: %s\n", categorias[escolha]);
        }

        // Mostra a carta do computador para a comparação
        printf("Carta do Computador:\n");
        mostrarCarta(cpu.cartas[0]);

        // Lógica para determinar o vencedor da rodada
        int valor_voce = voce.cartas[0].atributos[escolha];
        int valor_cpu = cpu.cartas[0].atributos[escolha];

        // Regra especial para o Peso (menor valor ganha)
        if (escolha == 2) { 
            if (valor_voce < valor_cpu) {
                vencedor_rodada = 1;
            } else if (valor_cpu < valor_voce) {
                vencedor_rodada = 0;
            } else {
                vencedor_rodada = -1;
            }
        } else { // Regra padrão (maior valor ganha)
            if (valor_voce > valor_cpu) {
                vencedor_rodada = 1;
            } else if (valor_cpu > valor_voce) {
                vencedor_rodada = 0;
            } else {
                vencedor_rodada = -1;
            }
        }
        
        // --- Conclusão da Rodada ---
        if (vencedor_rodada == 1) { // Você venceu
            printf("\nParabens! Voce ganhou a rodada!\n");
            moverCarta(&voce, &cpu);
            
            // Pega as cartas do pote se houver
            for(int i = 0; i < tamanho_pote; i++) {
                voce.cartas[voce.total++] = pote[i];
            }
            tamanho_pote = 0;
            
            turno_jogador = 1; // Mantém o turno com o jogador
        } else if (vencedor_rodada == 0) { // CPU venceu
            printf("\nQue pena! O computador ganhou a rodada!\n");
            moverCarta(&cpu, &voce);
            
            // Pega as cartas do pote se houver
            for(int i = 0; i < tamanho_pote; i++) {
                cpu.cartas[cpu.total++] = pote[i];
            }
            tamanho_pote = 0;
            
            turno_jogador = 0; // Turno passa para o computador
        } else { // Empate
            gerenciarEmpate(&voce, &cpu, pote, &tamanho_pote);
            // O turno permanece com quem jogou por ultimo
        }
        
        // Pausa para continuar a próxima rodada
        printf("\n");
        system("pause"); // Substitui o 'getchar'
    }

    // Fim do jogo
    printf("\n--- FIM DE JOGO ---\n");
    if (voce.total > 0) {
        printf("Parabens, voce venceu o jogo!\n");
    } else {
        printf("Voce perdeu! O computador venceu.\n");
    }

    return 0;
}
