#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

#define MAX_PRODUTOS 100
#define ARQUIVO_DADOS "estoque.dat"
#define NIVEL_MINIMO 5

typedef struct {
    int codigo;
    char nome[50];
    float preco;
    int quantidade;
    int minimo;
} Produto;

Produto estoque[MAX_PRODUTOS];
int totalProdutos = 0;

void limparBuffer() {
    while (getchar() != '\n');
}

void salvarDados() {
    FILE *arquivo = fopen(ARQUIVO_DADOS, "wb");
    if (arquivo == NULL) {
        printf("Erro ao abrir arquivo para escrita!\n");
        return;
    }
    
    fwrite(&totalProdutos, sizeof(int), 1, arquivo);
    fwrite(estoque, sizeof(Produto), totalProdutos, arquivo);
    
    fclose(arquivo);
}

void carregarDados() {
    FILE *arquivo = fopen(ARQUIVO_DADOS, "rb");
    if (arquivo == NULL) {
        printf("Arquivo de dados não encontrado. Criando novo...\n");
        return;
    }
    
    fread(&totalProdutos, sizeof(int), 1, arquivo);
    fread(estoque, sizeof(Produto), totalProdutos, arquivo);
    
    fclose(arquivo);
}

int encontrarProdutoPorCodigo(int codigo) {
    for (int i = 0; i < totalProdutos; i++) {
        if (estoque[i].codigo == codigo) {
            return i;
        }
    }
    return -1;
}

void cadastrarProduto() {
    if (totalProdutos >= MAX_PRODUTOS) {
        printf("Limite de produtos atingido!\n");
        return;
    }
    
    Produto novo;
    
    printf("\n--- Cadastro de Produto ---\n");
    
    printf("Código: ");
    scanf("%d", &novo.codigo);
    
    if (encontrarProdutoPorCodigo(novo.codigo) != -1) {
        printf("Já existe um produto com este código!\n");
        return;
    }
    
    printf("Nome: ");
    limparBuffer();
    fgets(novo.nome, 50, stdin);
    novo.nome[strcspn(novo.nome, "\n")] = '\0';
    
    printf("Preço: ");
    scanf("%f", &novo.preco);
    
    printf("Quantidade inicial: ");
    scanf("%d", &novo.quantidade);
    
    printf("Quantidade mínima: ");
    scanf("%d", &novo.minimo);
    
    estoque[totalProdutos] = novo;
    totalProdutos++;
    
    salvarDados();
    printf("Produto cadastrado com sucesso!\n");
}

void listarProdutos() {
    printf("\n--- Lista de Produtos ---\n");
    printf("%-6s %-30s %-10s %-8s %-8s\n", 
           "Código", "Nome", "Preço", "Quant.", "Mínimo");
    
    for (int i = 0; i < totalProdutos; i++) {
        printf("%-6d %-30s R$%-8.2f %-8d %-8d\n", 
               estoque[i].codigo, 
               estoque[i].nome, 
               estoque[i].preco, 
               estoque[i].quantidade,
               estoque[i].minimo);
    }
}

void editarProduto() {
    int codigo;
    printf("\n--- Editar Produto ---\n");
    printf("Digite o código do produto: ");
    scanf("%d", &codigo);
    
    int pos = encontrarProdutoPorCodigo(codigo);
    if (pos == -1) {
        printf("Produto não encontrado!\n");
        return;
    }
    
    printf("\nEditando produto: %s\n", estoque[pos].nome);
    printf("Novo nome (atual: %s): ", estoque[pos].nome);
    limparBuffer();
    fgets(estoque[pos].nome, 50, stdin);
    estoque[pos].nome[strcspn(estoque[pos].nome, "\n")] = '\0';
    
    printf("Novo preço (atual: %.2f): ", estoque[pos].preco);
    scanf("%f", &estoque[pos].preco);
    
    printf("Nova quantidade (atual: %d): ", estoque[pos].quantidade);
    scanf("%d", &estoque[pos].quantidade);
    
    printf("Novo mínimo (atual: %d): ", estoque[pos].minimo);
    scanf("%d", &estoque[pos].minimo);
    
    salvarDados();
    printf("Produto atualizado com sucesso!\n");
}

void excluirProduto() {
    int codigo;
    printf("\n--- Excluir Produto ---\n");
    printf("Digite o código do produto: ");
    scanf("%d", &codigo);
    
    int pos = encontrarProdutoPorCodigo(codigo);
    if (pos == -1) {
        printf("Produto não encontrado!\n");
        return;
    }
    
    printf("Tem certeza que deseja excluir %s? (S/N): ", estoque[pos].nome);
    limparBuffer();
    char confirmacao = toupper(getchar());
    
    if (confirmacao == 'S') {
        for (int i = pos; i < totalProdutos - 1; i++) {
            estoque[i] = estoque[i + 1];
        }
        totalProdutos--;
        salvarDados();
        printf("Produto excluído com sucesso!\n");
    } else {
        printf("Operação cancelada.\n");
    }
}

void movimentarEstoque(int entrada) {
    int codigo, quantidade;
    
    printf("\n--- %s Estoque ---\n", entrada ? "Entrada no" : "Saída do");
    printf("Digite o código do produto: ");
    scanf("%d", &codigo);
    
    int pos = encontrarProdutoPorCodigo(codigo);
    if (pos == -1) {
        printf("Produto não encontrado!\n");
        return;
    }
    
    printf("Produto: %s\n", estoque[pos].nome);
    printf("Quantidade atual: %d\n", estoque[pos].quantidade);
    printf("Digite a quantidade para %s: ", entrada ? "entrada" : "saída");
    scanf("%d", &quantidade);
    
    if (quantidade <= 0) {
        printf("Quantidade inválida!\n");
        return;
    }
    
    if (!entrada && quantidade > estoque[pos].quantidade) {
        printf("Quantidade insuficiente em estoque!\n");
        return;
    }
    
    if (entrada) {
        estoque[pos].quantidade += quantidade;
    } else {
        estoque[pos].quantidade -= quantidade;
    }
    
    salvarDados();
    printf("Estoque atualizado. Nova quantidade: %d\n", estoque[pos].quantidade);
}

void relatorioBaixoEstoque() {
    printf("\n--- Relatório de Baixo Estoque ---\n");
    printf("Produtos com quantidade abaixo do mínimo:\n");
    printf("%-6s %-30s %-8s %-8s %-8s\n", 
           "Código", "Nome", "Quant.", "Mínimo", "Faltam");
    
    int algumProduto = 0;
    
    for (int i = 0; i < totalProdutos; i++) {
        if (estoque[i].quantidade < estoque[i].minimo) {
            printf("%-6d %-30s %-8d %-8d %-8d\n", 
                   estoque[i].codigo, 
                   estoque[i].nome, 
                   estoque[i].quantidade,
                   estoque[i].minimo,
                   estoque[i].minimo - estoque[i].quantidade);
            algumProduto = 1;
        }
    }
    
    if (!algumProduto) {
        printf("Nenhum produto com estoque abaixo do mínimo.\n");
    }
}

void menuPrincipal() {
    printf("\n=== SISTEMA DE GERENCIAMENTO DE ESTOQUE ===\n");
    printf("1. Cadastrar produto\n");
    printf("2. Listar produtos\n");
    printf("3. Editar produto\n");
    printf("4. Excluir produto\n");
    printf("5. Entrada no estoque\n");
    printf("6. Saída do estoque\n");
    printf("7. Relatório de baixo estoque\n");
    printf("0. Sair\n");
    printf("Escolha uma opção: ");
}

int main() {
    carregarDados();
    
    int opcao;
    do {
        menuPrincipal();
        scanf("%d", &opcao);
        
        switch (opcao) {
            case 1:
                cadastrarProduto();
                break;
            case 2:
                listarProdutos();
                break;
            case 3:
                editarProduto();
                break;
            case 4:
                excluirProduto();
                break;
            case 5:
                movimentarEstoque(1);
                break;
            case 6:
                movimentarEstoque(0);
                break;
            case 7:
                relatorioBaixoEstoque();
                break;
            case 0:
                printf("Saindo do sistema...\n");
                break;
            default:
                printf("Opção inválida!\n");
        }
    } while (opcao != 0);
    
    return 0;
}
