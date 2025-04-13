#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

// Definições de constantes
#define MAX_PRODUTOS 100          // Número máximo de produtos no estoque
#define ARQUIVO_DADOS "estoque.dat" // Nome do arquivo de armazenamento
#define NIVEL_MINIMO 5            // Nível mínimo padrão para alertas

// Estrutura que representa um produto
typedef struct {
    int codigo;         // Código único do produto
    char nome[50];      // Nome do produto
    float preco;        // Preço unitário
    int quantidade;     // Quantidade em estoque
    int minimo;         // Quantidade mínima desejada
} Produto;

// Variáveis globais
Produto estoque[MAX_PRODUTOS];  // Array que armazena todos os produtos
int totalProdutos = 0;          // Contador de produtos cadastrados

// Função para limpar o buffer de entrada
void limparBuffer() {
    while (getchar() != '\n');
}

// Função para salvar os dados no arquivo
void salvarDados() {
    // Abre o arquivo em modo de escrita binária
    FILE *arquivo = fopen(ARQUIVO_DADOS, "wb");
    if (arquivo == NULL) {
        printf("Erro ao abrir arquivo para escrita!\n");
        return;
    }
    
    // Primeiro escreve o total de produtos
    fwrite(&totalProdutos, sizeof(int), 1, arquivo);
    // Depois escreve o array de produtos
    fwrite(estoque, sizeof(Produto), totalProdutos, arquivo);
    
    // Fecha o arquivo
    fclose(arquivo);
}

// Função para carregar os dados do arquivo
void carregarDados() {
    // Abre o arquivo em modo de leitura binária
    FILE *arquivo = fopen(ARQUIVO_DADOS, "rb");
    if (arquivo == NULL) {
        printf("Arquivo de dados não encontrado. Criando novo...\n");
        return;
    }
    
    // Primeiro lê o total de produtos
    fread(&totalProdutos, sizeof(int), 1, arquivo);
    // Depois lê o array de produtos
    fread(estoque, sizeof(Produto), totalProdutos, arquivo);
    
    // Fecha o arquivo
    fclose(arquivo);
}

// Função para encontrar um produto pelo código
int encontrarProdutoPorCodigo(int codigo) {
    // Percorre o array de produtos
    for (int i = 0; i < totalProdutos; i++) {
        if (estoque[i].codigo == codigo) {
            return i; // Retorna a posição se encontrar
        }
    }
    return -1; // Retorna -1 se não encontrar
}

// Função para cadastrar um novo produto
void cadastrarProduto() {
    // Verifica se há espaço no estoque
    if (totalProdutos >= MAX_PRODUTOS) {
        printf("Limite de produtos atingido!\n");
        return;
    }
    
    Produto novo; // Variável temporária para o novo produto
    
    printf("\n--- Cadastro de Produto ---\n");
    
    // Solicita e valida o código
    printf("Código: ");
    scanf("%d", &novo.codigo);
    
    // Verifica se o código já existe
    if (encontrarProdutoPorCodigo(novo.codigo) != -1) {
        printf("Já existe um produto com este código!\n");
        return;
    }
    
    // Solicita o nome do produto
    printf("Nome: ");
    limparBuffer(); // Limpa o buffer antes de ler strings
    fgets(novo.nome, 50, stdin);
    novo.nome[strcspn(novo.nome, "\n")] = '\0'; // Remove a quebra de linha
    
    // Solicita o preço
    printf("Preço: ");
    scanf("%f", &novo.preco);
    
    // Solicita a quantidade inicial
    printf("Quantidade inicial: ");
    scanf("%d", &novo.quantidade);
    
    // Solicita a quantidade mínima
    printf("Quantidade mínima: ");
    scanf("%d", &novo.minimo);
    
    // Adiciona o novo produto ao estoque
    estoque[totalProdutos] = novo;
    totalProdutos++;
    
    // Salva os dados no arquivo
    salvarDados();
    printf("Produto cadastrado com sucesso!\n");
}

// Função para listar todos os produtos
void listarProdutos() {
    printf("\n--- Lista de Produtos ---\n");
    // Cabeçalho da tabela
    printf("%-6s %-30s %-10s %-8s %-8s\n", 
           "Código", "Nome", "Preço", "Quant.", "Mínimo");
    
    // Imprime cada produto formatado
    for (int i = 0; i < totalProdutos; i++) {
        printf("%-6d %-30s R$%-8.2f %-8d %-8d\n", 
               estoque[i].codigo, 
               estoque[i].nome, 
               estoque[i].preco, 
               estoque[i].quantidade,
               estoque[i].minimo);
    }
}

// Função para editar um produto existente
void editarProduto() {
    int codigo;
    printf("\n--- Editar Produto ---\n");
    printf("Digite o código do produto: ");
    scanf("%d", &codigo);
    
    // Encontra a posição do produto
    int pos = encontrarProdutoPorCodigo(codigo);
    if (pos == -1) {
        printf("Produto não encontrado!\n");
        return;
    }
    
    // Permite editar cada campo
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
    
    // Salva as alterações
    salvarDados();
    printf("Produto atualizado com sucesso!\n");
}

// Função para excluir um produto
void excluirProduto() {
    int codigo;
    printf("\n--- Excluir Produto ---\n");
    printf("Digite o código do produto: ");
    scanf("%d", &codigo);
    
    // Encontra a posição do produto
    int pos = encontrarProdutoPorCodigo(codigo);
    if (pos == -1) {
        printf("Produto não encontrado!\n");
        return;
    }
    
    // Confirmação da exclusão
    printf("Tem certeza que deseja excluir %s? (S/N): ", estoque[pos].nome);
    limparBuffer();
    char confirmacao = toupper(getchar());
    
    if (confirmacao == 'S') {
        // Remove o produto deslocando os elementos seguintes
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

// Função para movimentar estoque (entrada ou saída)
void movimentarEstoque(int entrada) {
    int codigo, quantidade;
    
    printf("\n--- %s Estoque ---\n", entrada ? "Entrada no" : "Saída do");
    printf("Digite o código do produto: ");
    scanf("%d", &codigo);
    
    // Encontra o produto
    int pos = encontrarProdutoPorCodigo(codigo);
    if (pos == -1) {
        printf("Produto não encontrado!\n");
        return;
    }
    
    printf("Produto: %s\n", estoque[pos].nome);
    printf("Quantidade atual: %d\n", estoque[pos].quantidade);
    printf("Digite a quantidade para %s: ", entrada ? "entrada" : "saída");
    scanf("%d", &quantidade);
    
    // Valida a quantidade
    if (quantidade <= 0) {
        printf("Quantidade inválida!\n");
        return;
    }
    
    // Para saída, verifica se há estoque suficiente
    if (!entrada && quantidade > estoque[pos].quantidade) {
        printf("Quantidade insuficiente em estoque!\n");
        return;
    }
    
    // Atualiza o estoque
    if (entrada) {
        estoque[pos].quantidade += quantidade;
    } else {
        estoque[pos].quantidade -= quantidade;
    }
    
    // Salva as alterações
    salvarDados();
    printf("Estoque atualizado. Nova quantidade: %d\n", estoque[pos].quantidade);
}

// Função para gerar relatório de baixo estoque
void relatorioBaixoEstoque() {
    printf("\n--- Relatório de Baixo Estoque ---\n");
    printf("Produtos com quantidade abaixo do mínimo:\n");
    printf("%-6s %-30s %-8s %-8s %-8s\n", 
           "Código", "Nome", "Quant.", "Mínimo", "Faltam");
    
    int algumProduto = 0; // Flag para verificar se há produtos com baixo estoque
    
    // Percorre todos os produtos
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

// Função que exibe o menu principal
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

// Função principal
int main() {
    // Carrega os dados do arquivo ao iniciar
    carregarDados();
    
    int opcao;
    do {
        menuPrincipal();
        scanf("%d", &opcao);
        
        // Processa a opção selecionada
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
                movimentarEstoque(1); // 1 indica entrada
                break;
            case 6:
                movimentarEstoque(0); // 0 indica saída
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
