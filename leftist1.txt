// leftist.cpp: определяет точку входа для консольного приложения.
//
/*Левосторонняя куча (англ. leftist heap) — двоичное левосторонее дерево 
(не обязательно сбалансированное), но с соблюдением порядка кучи (heap order).*/

#include "stdafx.h"
#include <iostream>
#include <cstdlib>
using namespace std;

//объявление Node Class 
class LeftistNode
{
public:
	int key; //ключ, элемент кучи
	LeftistNode *left;
	LeftistNode *right;
	int dist; //расстояние от вершины до ближайшей свободной позиции в поддереве
	LeftistNode(int &key, LeftistNode *lt = NULL, 
		LeftistNode *rt = NULL, int np = 0)
	{
		this->key = key;
		right = rt;
		left = lt,
			dist = np;
	}
};

//Объявление Class 
class LeftistHeap
{
public:
	LeftistHeap();
	LeftistHeap(LeftistHeap &rhs);
	~LeftistHeap();
	int &findMin(); 
	void Insert(int &x); 
	void extractMin(); 
	void deleteMin(int &min); 
	void Merge(LeftistHeap &rhs);
	bool isEmpty();
	bool isFull();
	void makeEmpty();
private:
	LeftistNode *root;
	LeftistNode *Merge(LeftistNode *h1, LeftistNode *h2); 
	LeftistNode *Merge1(LeftistNode *h1, LeftistNode *h2);
	void swapChildren(LeftistNode *t);
	void restoreMemory(LeftistNode *t);
};

//Построение левостронней кучи
LeftistHeap::LeftistHeap()
{
	root = NULL;
}

LeftistHeap::LeftistHeap(LeftistHeap &rhs)
{
	root = NULL;
	*this = rhs;
}

// Опустошение кучи
LeftistHeap::~LeftistHeap()
{
	makeEmpty();
}

/*слияние переменной rhs. */
void LeftistHeap::Merge(LeftistHeap &rhs)
{
	if (this == &rhs)
		return;
	root = Merge(root, rhs.root);
	rhs.root = NULL;
}

/* слияние двух куч (вызов рекурсивного merge1)*/
LeftistNode *LeftistHeap::Merge(LeftistNode *h1, LeftistNode *h2)
{
	if (h1 == NULL)
		return h2;
	if (h2 == NULL)
		return h1;
	if (h1->key < h2->key) 
		return Merge1(h1, h2);
	else
		return Merge1(h2, h1);
}

/* слияние двух куч.
Предполагается, что деревья не пусты, а корень h1 содержит наименьший элемент.*/
LeftistNode *LeftistHeap::Merge1(LeftistNode *h1, LeftistNode *h2)
{
	if (h1->left == NULL)
		h1->left = h2;
	else
	{
		h1->right = Merge(h1->right, h2); //Пойдем направо и сольем правое поддерево с корнем h2 (корень второго дерева)
		if (h1->left->dist < h1->right->dist) //Могло возникнуть  нарушение левостороннести кучи. dist — расстояние от вершины до ближайшей свободной позиции в ее поддереве (dist.left>dist.right) 
			swapChildren(h1); //(меняем левого и правого ребенка)
		h1->dist = h1->right->dist + 1; //пересчитаем расстояние до ближайшей свободной позиции
	}
	return h1; //Каждый раз идем из уже существующей вершины только в правое поддерево
}

// Меняем двух детей 
void LeftistHeap::swapChildren(LeftistNode *t)
{
	LeftistNode *tmp = t->left;
	t->left = t->right;
	t->right = tmp;
}

/* Вставка элемента x в очередь приоритетов, поддерживая порядок кучи.
создаем дерево с одним ключом и вызываем merge*/
void LeftistHeap::Insert(int &x)
{
	root = Merge(new LeftistNode(x), root);
}

/* Найти наименьший элемент в очереди приоритетов. Минимум хранится в корне*/
int &LeftistHeap::findMin()
{
	return root->key;
}

/*извлечение минимального через merge (удалив root)
Извлекаем минимальное значение из корня, удаляем корень, сливаем левое и правое поддерево*/
void LeftistHeap::extractMin()
{
	LeftistNode *oldRoot = root;
	root = Merge(root->left, root->right);
	delete oldRoot;
}

/* Удалить наименьший элемент из очереди приоритетов*/
void LeftistHeap::deleteMin(int &min)
{
	if (isEmpty())
	{
		cout << "Heap is Empty" << endl;
		return;
	}
	min = findMin();
	extractMin();
}

/* Проверяет, что очередь приоритетов пуста.
Возвращает true, если пусто, false иначе*/
bool LeftistHeap::isEmpty()
{
	return root == NULL;
}

/* Проверяет, если очередь приоритетов логически заполнена.
Возвращает false в этой реализации.*/
bool LeftistHeap::isFull()
{
	return false;
}

// Сделать очередь пустой
void LeftistHeap::makeEmpty()
{
	restoreMemory(root); //восстановить память
	root = NULL;
}

//метод, чтобы сделать дерево пустым
void LeftistHeap::restoreMemory(LeftistNode * t)
{
	if (t != NULL)
	{
		restoreMemory(t->left);
		restoreMemory(t->right);
		delete t;
	}
}

int main()
{
	LeftistHeap h; //общая куча
	LeftistHeap h1; //первая
	LeftistHeap h2; //вторая
	cout << "Keys inserted in heaps:" << endl<<endl;
	for (int i = 0; i < 20; i++)
	{ 
		if (i % 2 == 0)
		{
			h.Insert(i);
			
			cout << i << " in Heap 1" << endl;
		}
		else
		{
			h1.Insert(i);
			cout << i << " in Heap 2" << endl;
		}
	}

	h.Merge(h1);
	h2 = h;
	cout <<endl<< "Keys deleted in heaps:" << endl <<endl;
	for (int i = 0; i < 20; i++)
	{
		int x;
		h2.deleteMin(x);
		cout << x << " Deleted" << endl;
		if (h2.isEmpty())
		{
			cout <<endl<<"The Heap is Empty" << endl<<endl;
			break;
		}
	}
	system("pause");
	return 0;
}

/*
int main()
{
LeftistHeap h;
LeftistHeap h1;
LeftistHeap h2;
int x;
int arr[] = { 1, 5, 7, 10, 15 };
int arr1[] = { 22, 75 };

h.Insert(arr[0]);
h.Insert(arr[1]);
h.Insert(arr[2]);
h.Insert(arr[3]);
h.Insert(arr[4]);
h1.Insert(arr1[0]);
h1.Insert(arr1[1]);

h.deleteMin(x);
cout << x << endl;

h1.deleteMin(x);
cout << x << endl;

h.Merge(h1);
h2 = h;

h2.deleteMin(x);
cout << x << endl;

system("pause");
return 0;

}*/


