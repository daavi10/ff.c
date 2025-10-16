# ff.c

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

// Struct do componente
typedef struct {
    char nome[30];
    char tipo[20];
    int prioridade;
} Componente;

#define MAX_COMPONENTES 20

// Protótipos
int bubbleSortNome(Componente[], int);
int insertionSortTipo(Componente[], int);
int selectionSortPrioridade(Componente[], int);
int buscaBinariaPorNome(Componente[], int, char[]);
void medirTempo(int (*algoritmo)(Componente[], int), Componente[], int, const char*);
void mostrarComponentes(Componente[], int);
void cadastrarComponentes();
void menu();

// Vetor global
Componente componentes[MAX_COMPONENTES];
int totalComponentes = 0;

// Mostrar componentes
void mostrarComponentes(Componente lista[], int n) {
    printf("\n📦 Componentes cadastrados (%d):\n", n);
    for (int i = 0; i < n; i++) {
        printf("%d. Nome: %s | Tipo: %s | Prioridade: %d\n",
               i + 1, lista[i].nome, lista[i].tipo, lista[i].prioridade);
    }
    printf("\n");
}

// Bubble sort por nome
int bubbleSortNome(Componente lista[], int n) {
    int comparacoes = 0;
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            comparacoes++;
            if (strcmp(lista[j].nome, lista[j + 1].nome) > 0) {
                Componente temp = lista[j];
                lista[j] = lista[j + 1];
                lista[j + 1] = temp;
            }
        }
    }
    return comparacoes;
}

// Insertion sort por tipo
int insertionSortTipo(Componente lista[], int n) {
    int comparacoes = 0;
    for (int i = 1; i < n; i++) {
        Componente chave = lista[i];
        int j = i - 1;
        while (j >= 0 && strcmp(lista[j].tipo, chave.tipo) > 0) {
            lista[j + 1] = lista[j];
            j--;
            comparacoes++;
        }
        if (j >= 0) comparacoes++;
        lista[j + 1] = chave;
    }
    return comparacoes;
}

// Selection sort por prioridade
int selectionSortPrioridade(Componente lista[], int n) {
    int comparacoes = 0;
    for (int i = 0; i < n - 1; i++) {
        int min = i;
        for (int j = i + 1; j < n; j++) {
            comparacoes++;
            if (lista[j].prioridade < lista[min].prioridade) {
                min = j;
            }
        }
        if (min != i) {
            Componente temp = lista[i];
            lista[i] = lista[min];
            lista[min] = temp;
        }
    }
    return comparacoes;
}

// Busca binária por nome
int buscaBinariaPorNome(Componente lista[], int n, char chave[]) {
    int inicio = 0, fim = n - 1, meio;
    while (inicio <= fim) {
        meio = (inicio + fim) / 2;
        int cmp = strcmp(lista[meio].nome, chave);
        if (cmp == 0)
            return meio;
        else if (cmp < 0)
            inicio = meio + 1;
        else
            fim = meio - 1;
    }
    return -1;
}

// Medir tempo e comparações
void medirTempo(int (*algoritmo)(Componente[], int), Componente lista[], int n, const char* descricao) {
    Componente copia[MAX_COMPONENTES];
    memcpy(copia, lista, sizeof(Componente) * n);

    clock_t inicio = clock();
    int comparacoes = algoritmo(copia, n);
    clock_t fim = clock();

    double tempo = (double)(fim - inicio) / CLOCKS_PER_SEC;

    printf("🛠️  Algoritmo: %s\n", descricao);
    printf("⏱️  Tempo de execução: %.6f segundos\n", tempo);
    printf("🔢 Comparações: %d\n", comparacoes);

    mostrarComponentes(copia, n);
}

// Cadastrar componentes
void cadastrarComponentes() {
    int quantidade;
    printf("Quantos componentes deseja cadastrar? (máx. %d): ", MAX_COMPONENTES);
    scanf("%d", &quantidade);
    getchar();

    if (quantidade < 1 || quantidade > MAX_COMPONENTES) {
        printf("❌ Quantidade inválida!\n");
        return;
    }

    for (int i = 0; i < quantidade; i++) {
        printf("\nComponente %d:\n", i + 1);

        printf("Nome: ");
        fgets(componentes[i].nome, sizeof(componentes[i].nome), stdin);
        componentes[i].nome[strcspn(componentes[i].nome, "\n")] = '\0';

        printf("Tipo: ");
        fgets(componentes[i].tipo, sizeof(componentes[i].tipo), stdin);
        componentes[i].tipo[strcspn(componentes[i].tipo, "\n")] = '\0';

        printf("Prioridade (1 a 10): ");
        scanf("%d", &componentes[i].prioridade);
        getchar();
    }

    totalComponentes = quantidade;
    printf("✅ Cadastro concluído!\n");
}

// Menu principal
void menu() {
    int opcao;
    char chaveBusca[30];

    do {
        printf("\n=== MENU DE MONTAGEM DA TORRE DE RESGATE ===\n");
        printf("1. Cadastrar componentes\n");
        printf("2. Ordenar por nome (Bubble Sort)\n");
        printf("3. Ordenar por tipo (Insertion Sort)\n");
        printf("4. Ordenar por prioridade (Selection Sort)\n");
        printf("5. Buscar componente-chave por nome (Busca Binária)\n");
        printf("6. Mostrar componentes cadastrados\n");
        printf("0. Sair\n");
        printf("Escolha uma opção: ");
        scanf("%d", &opcao);
        getchar();

        switch (opcao) {
            case 1:
                cadastrarComponentes();
                break;
            case 2:
                medirTempo(bubbleSortNome, componentes, totalComponentes, "Bubble Sort (Nome)");
                break;
            case 3:
                medirTempo(insertionSortTipo, componentes, totalComponentes, "Insertion Sort (Tipo)");
                break;
            case 4:
                medirTempo(selectionSortPrioridade, componentes, totalComponentes, "Selection Sort (Prioridade)");
                break;
            case 5: {
                if (totalComponentes == 0) {
                    printf("❌ Nenhum componente cadastrado!\n");
                    break;
                }
                printf("Digite o nome do componente-chave para busca: ");
                fgets(chaveBusca, sizeof(chaveBusca), stdin);
                chaveBusca[strcspn(chaveBusca, "\n")] = '\0';

                bubbleSortNome(componentes, totalComponentes); // garantir ordenação
                int posicao = buscaBinariaPorNome(componentes, totalComponentes, chaveBusca);

                if (posicao != -1) {
                    printf("🔓 Componente-chave encontrado!\n");
                    printf("Nome: %s | Tipo: %s | Prioridade: %d\n",
                           componentes[posicao].nome,
                           componentes[posicao].tipo,
                           componentes[posicao].prioridade);
                } else {
                    printf("❌ Componente-chave NÃO encontrado.\n");
                }
                break;
            }
            case 6:
                mostrarComponentes(componentes, totalComponentes);
                break;
            case 0:
                printf("🏁 Encerrando montagem da torre. Boa sorte na fuga!\n");
                break;
            default:
                printf("❌ Opção inválida. Tente novamente.\n");
        }
    } while (opcao != 0);
}

// Função principal
int main() {
    menu();
    return 0;
}
