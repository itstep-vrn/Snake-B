# Игра "Змейка"

---
Консольная игра "Змейка", где главная цель собрать как можно больше очков. Существует три уровня сложности, один из которых игрок выбирает до начала игры. С повышением уровня повышается скорость змеи и количество препядствий. При столкновении со стенкой, камнем или собственным хвостом игра оканчивается, а при подборе фрукта увеличивается размер змеи и количество очков. Управление змейкой осуществляется клавишами 'w', 'a', 's', 'd', но это только изменяет направление так как змейка в постоянном движении.

---


### В программе используются следующие глобальнае переменные и константы:

`const int play_field_width = 20;` - ширина поля

`const int play_field_height = 20;` - высота поля

`const int size_of_field = play_field_width * play_field_height;` - общий размер массива

`int play_field[size_of_field];` - массив игрового поля

#### Информация о змейке:

`int head_x_position;` - x координата головы змейки (на самом деле оспользуется для подсчета нужного элемента массива так как тот является одномерным, то же самое с у координатой)

`int head_y_position;`- у координата головы змейки

`int direction = 1;` - переменная определяющая направление змейки

`int snake_length = 3;` - длина змеи

`int score = 0;` - переменная для посчета набранных очков

`int difficulty;` - переменная, по которой при вводе определяется сложность игры

#### Использованные библиотеки
```cpp
#include <iostream>
#include <conio.h>
#include <windows.h>
#include <time.h>
```

---
Основной функцией является `run()`, в которой реализуется игра (с использованием других подфункций). 
```cpp
void run()
{
	// создание поля
	initialize_play_field();
	game_running = true;
	while (game_running)
	{
		//проверка на нажатие клавиши и получение её значения для изменения направления змейки
		if (_kbhit())
		{
			char key = _getch();
			switch (key) {
			case 'w':
				if (direction != 3) direction = 1;
				break;
			case 'd':
				if (direction != 4) direction = 2;
				break;
			case 's':
				if (direction != 1) direction = 3;
				break;
			case 'a':
				if (direction != 2) direction = 4;
				break;
			}
		}
		// обновить поле
		update_field();
		system("cls");
		// вывод поля
		print_play_field();
		//определение скорости змейки по выбранной сложности
		switch (difficulty)
		{
		case 1:Sleep(400);
			break;
		case 2:Sleep(300);
			break;
		case 3:Sleep(200);
			break;
		}
	}

	// конец игры
	cout << "!-----Game over-----!" << endl << "Your score is: " << score;
}

```

---

В начале идет инициализация поля без вывода через функцию `initialize_play_field()`, в которой голова змейки помещается в середину игрового поля и генерируется еда:

```cpp
void initialize_play_field()
{
	// начало змеи в середине поля (play_field[210])
	head_x_position = play_field_width / 2;
	head_y_position = play_field_height / 2;
	play_field[head_x_position + head_y_position * play_field_height] = 1;
	// левая и правая стенка

	for (int x = 0; x < play_field_width; x++)
	{    //берет play_field[0] и play_field [380], одновременно заполняет -1 и тд, до play_field[19] и play_field [399], т.е верхняя и нижняя строка
		play_field[x] = -1;
		play_field[x + (play_field_height - 1) * play_field_width] = -1;
	}

	// верхняя и нижняя стена
	for (int y = 0; y < play_field_height; y++)
	{    //тоже самое но берет play_field[0] и play_field[19], затем play_field[20] и play_field[39] т.е. пропуская 18 "игровых полей" с каждой итерацией
		play_field[0 + y * play_field_height] = -1;
		play_field[(play_field_width - 1) + y * play_field_width] = -1;
	}

	// создание изначальной еды
	generate_food();

```

---

Еда генерируется с помощью `generate_food()` путем подбора случайных координат исключая границы поля (функция `generate_rock()` работает ровно так же но присваевается `-3` вместо `-2`):
```cpp
void generate_food() {
	int x = 0;
	int y = 0;
	do
	{

		srand(time(NULL));
		x = rand() % (play_field_width - 2) + 1;
		y = rand() % (play_field_height - 2) + 1;

		// проверка на свободность поля
	} while (play_field[x + y * play_field_width] != 0);

	// положить еду
	play_field[x + y * play_field_width] = -2;
}
```
---

Затем игра переходит в цикл, окончание которого определено столкновением змейки со стеной или своим хвостом. 
Поле постоянно обновляется функцией `update_field()`:
```cpp
void update_field()
{
	// движение
	switch (direction) {
	case 1: move_snake(-1, 0);     //1:вверх
		break;
	case 2: move_snake(0, 1);	//2:право		
		break;
	case 3: move_snake(1, 0);	//3:вниз
		break;
	case 4: move_snake(0, -1);	//4:лево
		break;
	}

	for (int i = 0; i < size_of_field; i++)
	{
		if (play_field[i] > 0) play_field[i]--;
	}

}
```

Путем постоянного уменьшения на 1 всех элементов массива, которые больше нуля, получается что хвост змеи всегда следует за головой.
Движение реальзуется в функции `move_snake()`:

---

```cpp
void move_snake(int x, int y) {
	//новые координаты головы
	int newx = head_x_position + x;
	int newy = head_y_position + y;
	// проверка на еду
	if (play_field[newx + newy * play_field_width] == -2)
	{
		// удлинение 
		snake_length++;
		score++;
		// создание новой еды
		generate_food();
	}

	// проверка на стену и собственный хвост
	else if (play_field[newx + newy * play_field_width] != 0)
	{
		game_running = false;
	}
	
	head_x_position = newx;
	head_y_position = newy;
	play_field[head_x_position + head_y_position * play_field_width] = snake_length + 1;
}

```
Здесь с помощью преданных значений в функции ```update_field()``` происходит подсчет новых координат головы змейки. Если не обнаружено
условия на проигрыыш, то подсчитаному элементу массива присваивается значение ```snake_length + 1```. Это делается так как сразу после записи данного значения на место головы змейки все элементы уменьшаются на один. Если не использовать это то размер змейки становится меньше нужного.

---

Функция `void print_play_field()` выводит игру на экран:
```cpp
void print_play_field()
{
	for (int x = 0; x < play_field_width; x++)
	{
		for (int y = 0; y < play_field_height; y++)
		{
			// значение в проверяемых x,y координатах
			std::cout << get_play_field_value(play_field[x + y * play_field_width]);
		}
		// вывод в виде поля
		cout << endl;

	}
}
```
Массив выводится построчно, но элементы индексируются сверху вниз, т.е. выводится каждый двадцатый элемент. Из за этого в функцию `move_snake()` передаются значения `(-1,0)` для движения вверх, вместо `(0,1)` как на обычной x,y плоскости (например индекс начальной кординаты `play_field[210]` будет уменьшаться или увеличиваться на `1` при движении головы змейки вверх или вниз, но при движении налево или направо индекс будет изменятся на `19`).

Функция использует `char get_play_field_value(int value)` для определения символа для вывода:

```cpp
char get_play_field_value(int value)
{
	find_max(play_field);
	// змейка
	if (direction == 1 && value == max && value != 0) return '^';
	if (direction == 2 && value == max && value != 0) return '>';
	if (direction == 3 && value == max && value != 0) return 'V';
	if (direction == 4 && value == max && value != 0) return '<';

	if (value > 0 && value < max) return '*';

	switch (value) {
		// стена
	case -1: return 'O';
		// еда
	case -2: return '@';
		//камень
	case -3:return 'X';
}
```

`find_max()` возвращает наибольший элемент элемент массива (голова змейки) и в соответсвии выводится стрелка с направлением движения змейки.

---
