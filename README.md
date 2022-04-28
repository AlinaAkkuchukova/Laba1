# Лабораторная работа № 1 по дисциплине "Инженерная и компьютерная графика"
## 1. Установка библиотек OpenGL
+ freeglut
+ glew
+ glm
## 2. Создание окна
```C++
  glutInit(&argc, argv);
  //инициализация GLUT
  glutInitDisplayMode(GLUT_DOUBLE | GLUT_RGBA);
  //установка двойной буферизации и буфера цвета
  glutInitWindowSize(1024, 768);
  //задание размеров окна
  glutInitWindowPosition(100, 100);
  //задание позиции окна
  glutCreateWindow("Tutorial 01"); 
  //задание имени окна
  glClearColor(0.0f, 0.0f, 0.0f, 0.0f);
  //задание цвета фона окна
  glutDisplayFunc(RenderSceneCB);
  //вызов функции отрисовки
  glutMainLoop(); 
  //запуск цикла GLUT
```
Функция отрисовки
```C++
 static void RenderSceneCB()
 {
    glClear(GL_COLOR_BUFFER_BIT);
    //очистка буфера кадра
    glutSwapBuffers();
    //функция просит GLUT поменять фоновый буфер и буфер кадра местами
}
```
## 3. Рисование точки
Глобальная переменная
```C++
GLuint VBO;
//указатель на буфер вершин
```
Добавим в main()
```C++
glm::vec3 Vertices[1];
//создание вектора вершины
Vertices[0] = glm::vec3(0.0f, 0.0f, 0.0f);
//создание вектора с координатами x, y, z 
glGenBuffers(1, &VBO);
//генерация объектов переменных типов
glBindBuffer(GL_ARRAY_BUFFER, VBO);
//буфер будет хранить массив вершин
glBufferData(GL_ARRAY_BUFFER, sizeof(Vertices), Vertices, GL_STATIC_DRAW);
//заполнение буфера
```
Добавим в RenderSceneCB()
```C++
glEnableVertexAttribArray(0);
//разрешим использование каждого атрибута вершины в конвейере
glBindBuffer(GL_ARRAY_BUFFER, VBO);
//обратно привязываем буфер, приготавливая его для отрисовки
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, 0);
//говорим конвейеру как воспринимать данные внутри буфера
glDrawArrays(GL_POINTS, 0, 1);
//вызов функции отрисовки
glDisableVertexAttribArray(0);
//отключение использования каждого атрибута вершины в конвейере
```
## 4. Рисование треугольника
Изменим создание массива векторов в main()
```C++
glm::vec3 Vertices[3];
Vertices[0] = glm::vec3(1.0f, 1.0f, 0.0f);
Vertices[1] = glm::vec3(-1.0f, 1.0f, 0.0f);
Vertices[2] = glm::vec3(0.0f, -1.0f, 0.0f);
//создание векторов с координатами x, y, z
```
Изменим режим рисования в RenderSceneCB()
```C++
glDrawArrays(GL_TRIANGLES, 0, 3);
```
