#include <stdio.h>
#include <stdlib.h>
#include <locale.h>
#include <string.h>
#define TAM 256

typedef struct no
{
    unsigned char c;
    int frequencia;
    struct no *esq, *dir, *prox;
} No;

typedef struct
{
    No *inicio;
    int tam;
} Lista;

//---------tABELA DE FREQUENCIA------------
void inicializa_tabela(unsigned int tab[])
{
    for (int i = 0; i < TAM; i++)
    {
        tab[i] = 0;
    }
}

void preenche_tab(unsigned char texto[], unsigned int tab[])
{
    int i = 0;
    while (texto[i] != '\0')
    {
        tab[texto[i]]++;
        i++;
    }
}

void imprime_tab(unsigned int tab[])
{
    printf("\t TABELA DE FREQUENCIA\n");
    for (int i = 0; i < TAM; i++)
    {
        if (tab[i] > 0)
            printf("\t %d = %d = %c\n", i, tab[i], i);
    }
}

//-----------LISTA-----------

void criar_lista(Lista *lista)
{
    lista->inicio = NULL;
    lista->tam = 0;
}

void inserir_ord(Lista *lista, No *no)
{
    No *aux, *ant;
    if (lista->inicio == NULL)
    {
        lista->inicio = no;
    }
    else if (no->frequencia < lista->inicio->frequencia)
    {
        no->prox = lista->inicio;
        lista->inicio = no;
    }
    else
    {
        aux = lista->inicio;
        while (aux && aux->frequencia <= no->frequencia)
        {
            ant = aux;
            aux = aux->prox;
        }
        ant->prox = no;
        no->prox = aux;
    }
    lista->tam++;
}

void preencher_lista(unsigned int tab[], Lista *lista)
{
    int i;
    No *novo;
    for (i = 0; i < TAM; i++)
    {
        if (tab[i] > 0)
        {
            novo = malloc(sizeof(No));
            if (novo)
            {
                novo->c = i;
                novo->frequencia = tab[i];
                novo->dir = NULL;
                novo->esq = NULL;
                novo->prox = NULL;

                inserir_ord(lista, novo);
            }
            else
            {
                printf("Erro ao alocar memoria em preencher lista!\n");
                break;
            }
        }
    }
}

void imprimir_lista(Lista *lista)
{
    No *aux = lista->inicio;

    printf("\n\tLista ordenada: Tamanho: %d\n", lista->tam);
    while (aux)
    {
        printf("\tCaracter: %c Frequencia: %d\n", aux->c, aux->frequencia);
        aux = aux->prox;
    }
}

//------------ARVORE---------------
No *remove_no(Lista *lista)
{
    No *aux = NULL;
    if (lista->inicio)
    {
        aux = lista->inicio;
        lista->inicio = aux->prox;
        aux->prox = NULL;
        lista->tam--;
    }
    return aux;
}

No *montar_arvore(Lista *lista)
{
    No *primeiro, *segundo, *novo;
    while (lista->tam > 1)
    {
        primeiro = remove_no(lista);
        segundo = remove_no(lista);
        novo = malloc(sizeof(No));
        if (novo)
        {
            novo->c = '+';
            novo->frequencia = primeiro->frequencia + segundo->frequencia;
            novo->esq = primeiro;
            novo->dir = segundo;
            novo->prox = NULL;

            inserir_ord(lista, novo);
        }
        else
        {
            printf("\n\tErro a alocar memoria em montaraarvore!\n");
            break;
        }
    }
    return lista->inicio;
}

void imprimir_arvore(No *raiz, int tam)
{
    if (raiz->esq == NULL && raiz->dir == NULL)
    {
        printf("\tFolha: %c  Altura: %d\n", raiz->c, tam);
    }
    else
    {
        imprimir_arvore(raiz->esq, tam + 1);
        imprimir_arvore(raiz->dir, tam + 1);
    }
}

//------------------MONTAR O DICIONÁRIO---------------

int altura_arv(No *raiz)
{
    int esq, dir;
    if (raiz == NULL)
    {
        return -1;
    }
    else
    {
        esq = altura_arv(raiz->esq) + 1;
        dir = altura_arv(raiz->dir) + 1;
        if (esq > dir)
            return esq;
        else
            return dir;
    }
}

char **aloca_dic(int coluna)
{
    char **dic;
    dic = malloc(sizeof(char *) * TAM);
    for (int i = 0; i < TAM; i++)
    {
        dic[i] = calloc(coluna, sizeof(char));
    }
    return dic;
}

void gerar_dic(char **dic, No *raiz, char *caminho, int colunas)
{
    char esq[colunas], dir[colunas];
    if (raiz->esq == NULL && raiz->dir == NULL)
        strcpy(dic[raiz->c], caminho);
    else
    {
        strcpy(esq, caminho);
        strcpy(dir, caminho);

        strcat(esq, "0");
        strcat(dir, "1");

        gerar_dic(dic, raiz->esq, esq, colunas);
        gerar_dic(dic, raiz->dir, dir, colunas);
    }
}

void imprime_dic(char **dic)
{
    printf("\tDicionario:\n");
    for (int i = 0; i < TAM; i++)
    {
        if (strlen(dic[i]) > 0)
            printf("\t%3d: %s\n", i, dic[i]);
    }
}

//-------------CODIFICAÇÃO----------------

int tam_string(char **dic, unsigned char *texto)
{
    int i = 0, tam = 0;
    while (texto[i] != '\0')
    {
        tam = tam + strlen(dic[texto[i]]);
        i++;
    }
    return tam + 1;
}

char *codificar(char **dic, unsigned char *texto)
{
    int tam = tam_string(dic, texto), i = 0;
    char *codigo = calloc(tam, sizeof(char));
    while (texto[i] != '\0')
    {
        strcat(codigo, dic[texto[i]]);
        i++;
    }
    return codigo;
}

//-------------DECODIFICAÇÃO---------------
char* decodificar(unsigned char texto[], No *raiz){
    int i = 0;
    No *aux = raiz;
    char temp[2];
    char *decodificado = calloc(strlen(texto),sizeof(char));
    while(texto[i] != '\0'){
        if(texto[i] == '0')
            aux = aux->esq;
        else
            aux = aux->dir;
        if(aux->esq == NULL && aux->dir == NULL){
            temp[0] = aux->c;
            temp[1] = '\0';
            strcat(decodificado,temp);
            aux = raiz;
        }
        i++;
    }
    return decodificado;
}


int main()
{
    unsigned char texto[] = "O principio mais forte do crescimento reside na escolha humana." ;
    unsigned int tabela_freq[TAM];
    Lista lista;
    No *arv;
    char **dic;
    int colunas;
    char *codificado,*decodificado;

    setlocale(LC_ALL, "Portuguese");

    inicializa_tabela(tabela_freq);
    preenche_tab(texto, tabela_freq);
    imprime_tab(tabela_freq);

    criar_lista(&lista);
    preencher_lista(tabela_freq, &lista);
    imprimir_lista(&lista);

    arv = montar_arvore(&lista);
    printf("\t\nArvore de Huffman:\n");
    imprimir_arvore(arv, 0);

    colunas = altura_arv(arv) + 1;
    dic = aloca_dic(colunas);
    gerar_dic(dic, arv, "", colunas);
    imprime_dic(dic);

    codificado = codificar(dic, texto);
    printf("\n\tTexto codificado: %s\n", codificado);

    decodificado = decodificar(codificado,arv);
    printf("\n\tTexto decodificado: %s\n",decodificado);


    return 0;
}
