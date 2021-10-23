# hw_flita_2
my second homework - checking a graph against a tree 

For the program to work, you need to create a file with the name graph and enter a graph in it in the format x--y, where x and y are adjacent vertices.



// ввести граф в формате списка смежных между собой вершин и проверить, является ли данный граф деревом
// визуализировать граф через что-нибудь
```c
#include <stdio.h>
#include <stdbool.h>
#include <stdlib.h>

struct line //структура смежных вершин
{
    int node1; // вершина 1
    int node2; // вершина 2, смежная вершине 1
};

struct node //структура уникальных вершин
{
    int number;   // вершина
    bool visited; // была ли она посещена (нужно при обходе в глубину при проверки графа на связность)
};

int length = 0, uniq_count = 0;

// считывание количества строк в прикреплённом файлике
size_t get_length(const char *filename)
{
    size_t size = 0;
    FILE *file;
    file = fopen(filename, "r");
    if ((file = fopen(filename, "r")) == NULL)
    {
        printf("Foner read error!\n");
        exit(1);
    }
    while (!feof(file))
    {
        if (fgetc(file) == '\n')
            size++;
    }
    size++;
    return size;
}

//заполнение массива структуры смежных вершин чиселками из прикреплённого файлика
void get_lines(struct line *lines, const char *filename)
{
    int l = 0;
    FILE *file;
    file = fopen(filename, "r");
    while (fscanf(file, "%d--%d", &(lines[l].node1), &(lines[l].node2)) != EOF)
    {
        l++;
    }
}

//проверка на простоту (мультиграф может содержать петли или кратные связи и не может быть деревом)
//проверка на наличие петель
bool loop_search(int l, struct line *lines)
{
    int i = 0;
    bool fl = 0;
    while ((i < l) && (fl == 0))
    {
        if (lines[i].node1 == lines[i].node2)
        {
            fl = 1;
            printf("The graph contains loops => the graph isn't a simple graph => the graph isn't a tree\n");
        }
        i++;
    }
    return fl;
}

//проверка на наличие кратных связей
bool double_bonds_search(int l, struct line *lines)
{
    int i = 0, j = 0;
    bool fl = 0;
    while ((i < l) && (fl == 0))
    {
        j = i + 1;
        while ((j < l) && (fl == 0))
        {
            if ((lines[i].node1 == lines[j].node1) && (lines[i].node2 == lines[j].node2))
            {
                fl = 1;
                printf("The graph contains double bonds => the graph isn't a simple graph => the graph isn't a tree\n");
            }
            else if ((lines[i].node1 == lines[j].node2) && (lines[i].node2 == lines[j].node1))
            {
                fl = 1;
                printf("The graph contains double bonds => the graph isn't a simple graph => the graph isn't a tree\n");
            }
            j++;
        }
        i++;
    }
    return fl;
}

//заполнение массива структур уникальных вершин
int get_uniq_node(int l, struct line *lines, struct node *nodes)
{
    int i = 0, c = 0; // с - количество уникальных вершин, т.е. длина массива структур уникальных вершин
    for (i = 0; i < l; i++)
    {
        int j;
        for (j = 0; j < c; j++)
        {
            if (lines[i].node1 == nodes[j].number)
                break;
        }
        if (j == c)
        {
            nodes[c].number = lines[i].node1;
            nodes[c].visited = 0;
            c++;
        }
    }
    for (i = 0; i < l; i++)
    {
        int j;
        for (j = 0; j < c; j++)
        {
            if (lines[i].node2 == nodes[j].number)
                break;
        }
        if ((j == c) && (lines[i].node2 != 0))
        {
            nodes[c].number = lines[i].node2;
            nodes[c].visited = 0;
            c++;
        }
    }
    return c;
}

//проверка на связность графа методом dsf
//поиск номера символа в массиве уникальных вершин по его значению
int return_index(struct node *nodes, int node)
{
    int i = 0;
    for (i = 0; i < uniq_count; i++)
        if (nodes[i].number == node)
        {
            return i;
            break;
        }
}

//поиск в глубину
void DFS(struct line *lines, struct node *nodes, int node) // node передаём как 0 чтобы начать рассматривать массив с нулевого элемента
{
    int i = 0, next_n = 0;
    nodes[node].visited = 1; //обозначаем вершину как посещенную
    // printf("Node %d marked visited\n",nodes[node].number);
    for (i = 0; i < length; i++)
    {
        if ((lines[i].node1 == nodes[node].number) && (lines[i].node2 != 0))
        {
            // printf("Node %d marked connected\n",lines[i].node2);
            next_n = return_index(nodes, lines[i].node2); //следующая вершина - смежная с только что помеченной
            if (nodes[next_n].visited == 0)               //если следующая еще не помеченна
                DFS(lines, nodes, next_n);                //вызываем рекурсию, обозначив номер следующей как вершину, с которой мы рассматриваем цикл
        }
    }
}

//по тому же принципу делаем проверку на наличие циклов
bool DFS1(struct line *lines, struct node *nodes, int node) // node передаём как 0 чтобы начать рассматривать массив с нулевого элемента
{
    bool cycle = 0;               
    int i = 0, next_n = 0;
    nodes[node].visited = 1; //обозначаем вершину как посещенную
    // printf("Node %d marked visited\n",nodes[node].number);
    for (i = 0; i < length; i++)
    {
        if (((lines[i].node1 == nodes[node].number) || (lines[i].node2 == nodes[node].number)) && (lines[i].node2 != 0))
        {
            // printf("Node %d marked connected\n",lines[i].node2);
            next_n = return_index(nodes, lines[i].node2); //следующая вершина - смежная с только что помеченной
            if (nodes[next_n].visited == 0)               //если следующая еще не помеченна
                DFS1(lines, nodes, next_n);               //вызываем рекурсию, обозначив номер следующей как вершину, с которой мы рассматриваем цикл
            else if (nodes[next_n].visited == 1)
            {
                cycle = 1;
                return cycle;
                printf(".cycle = \n", cycle);
                exit(1);
            }
        }
    }
    return cycle;
}

//создание и заполнение Dotfile
void create_dot_file(const char *from_file, const char *to_file)
{
    char c;
    FILE *from, *to;
    from = fopen(from_file, "r"); // открыли файл Dotfile.txt для записи
    to = fopen(to_file, "w");
    if (
        (to = fopen("Dotfile.gv", "w+")) == NULL)
    {
        printf("Foner read error!\n"); // если ошибка, то печатаем ошибку
        exit(1);
    }
    else

    fputs("graph G{\n", to);

    while ((c = getc(from)) != EOF)
    {
        if (c == '\n')
            putc(';', to);
        putc(c, to);
    }

    fputs(";\n}", to);

    fclose(to);
    fclose(from);
}

int main(void)
{
    int i = 0;
    length = get_length("graph");
    bool fl1 = 0, fl2 = 0;

    struct line list_lines[length]; // соездаем массив структур с длинной равной количеству строк в файле
    get_lines(list_lines, "graph"); // заполняем его

    //   вывод для проверки
    printf("Тhis graph was read from the file 'graph' \n");
    for (i = 0; i < length; i++)
    {
        printf("%d--%d\n", list_lines[i].node1, list_lines[i].node2);
    }

    //ПРОВЕРКА НА ПРОСТОТУ
    fl1 = loop_search(length, list_lines);         // проверка на петли
    fl2 = double_bonds_search(length, list_lines); // проверка на кратные связи
    if ((fl1 == 0) && (fl2 == 0))                  // не делаем лишнюю лаботу и проверяем на связность только если граф простой
    {
        printf("The graph is a simple graph\n");

        // создаём массив структур уникальных вершин
        struct node uniq_nodes[2 * length];                         // длиной 2n т.к. это максимально возможная длина если все вершины разные
        uniq_count = get_uniq_node(length, list_lines, uniq_nodes); //заполняем его
        //выводим его
        printf("Unique vertices of this graph\n");
        for (i = 0; i < uniq_count; i++)
        {
            printf("%d ", uniq_nodes[i].number);
        }
        printf("\n");

        //проверка на связность
        DFS(list_lines, uniq_nodes, 0);
        bool is_connected = 1;

        printf("Vizited vertices marked '1'\n");
        for (i = 0; i < uniq_count; i++)
        {
            printf("%d ", uniq_nodes[i].visited);
            //если после конъюнкции с булевыми значениями,
            //помеченными как посещённые (1) или не посещённые (0),
            // is_connected останется единицей, то мы посетили все вершины => граф связный
            is_connected = is_connected * uniq_nodes[i].visited;
        }
        printf("\n");

        if (is_connected) //если граф простой
        {
            printf("This graph is connected!\n");

            //проверка на наличие циклов
            for (i = 0; i < uniq_count; i++)
                uniq_nodes[i].visited = 0;
            if (DFS1(list_lines, uniq_nodes, 0) == 1)
                printf("This graph contains cycle => Тhe graph isn't a tree\n");
            else
                printf("Тhe graph is a tree!!!!!!!!!!!!!!!!!\n");
        }
        else
        {
            printf("This graph is not connected! Тhe graph isn't a tree\n");
        }
    }

    //визуализация

    //создадим и заполним файл Dotfile.txt
    create_dot_file("graph", "Dotfile.gv");

    return 0;
}
```
