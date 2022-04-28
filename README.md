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

# Лабораторная работа № 2 по дисциплине "Инженерная и компьютерная графика"
## 1. Перемещение
```C++
GLuint gWorldLocation;
//указатель для доступа к всемирной матрице
```
Добавим код шейдера
```C++
#version 330
// Входной тип данных - vec3 в позиции 0
layout (location = 0) in vec3 Position;
// uniform-переменная типа mat4
uniform mat4 gWorld;
void main() {
gl_Position = gWorld * vec4(Position, 1.0);
// Умножаем вектор вершин на всемирную матрицу для смещения треугольника
}
```
Добавим в функцию отрисовки
```C++
static float Scale = 0.0f;
//переменная для изменения значения X
Scale += 0.001f;
//с каждой отрисовкой увеличиваем Scale
```
Подготовим матрицу 4х4
```C++
glm::mat4 World;
World[0][0] = 1.0f; World[0][1] = 0.0f; World[0][2] = 0.0f; World[0][3] = sinf(Scale);
World[1][0] = 0.0f; World[1][1] = 1.0f; World[1][2] = 0.0f; World[1][3] = 0.0f;
World[2][0] = 0.0f; World[2][1] = 0.0f; World[2][2] = 1.0f; World[2][3] = 0.0f;
World[3][0] = 0.0f; World[3][1] = 0.0f; World[3][2] = 0.0f; World[3][3] = 1.0f;
glUniformMatrix4fv(gWorldLocation, 1, GL_TRUE, &World.m[0][0]);
//загружаем данные в uniform-переменные шейдера
```
Добавим новые функции
```C++
void addShader(GLuint shaderProgram, const std::string pShaderText, GLenum shaderType) {
//функция, добавляющая шейдер к программе
GLuint shaderObj = glCreateShader(shaderType);
//создание шейдера
if (shaderObj == 0) {
std::cerr « "Error: creating shader type " « shaderType « std::endl;
exit(0);
}
//сохраним код шейдера в массиве
const GLchar *p[1];
p[0] = pShaderText.c_str();

//массив длин кодов шейдеров
GLint lengths[1];
lengths[0] = pShaderText.length();
glShaderSource(shaderObj, 1, p, lengths);
//задание исходников шейдера
glCompileShader(shaderObj);

//проверим, что шейдер успешно скомпилировался
GLint success;
glGetShaderiv(shaderObj, GL_COMPILE_STATUS, &success);
if (!success) {
GLchar infoLog[1024];
glGetShaderInfoLog(shaderObj, sizeof(infoLog), nullptr, infoLog);
std::cerr « "Error compiling shader type " « shaderType « ": '" « infoLog « "'" « std::endl;
exit(1);
}

//добавим шейдер в программу
glAttachShader(shaderProgram, shaderObj);
}

//функция, компилирующая программу-шейдер
void compileShaders() {
//создадим программу-шейдер
GLuint shaderProgram = glCreateProgram();
if (shaderProgram == 0) {
std::cerr « "Error creating shader program" « std::endl;
exit(1);
}

//добавим шейдер для вершин
addShader(shaderProgram, pVS, GL_VERTEX_SHADER);
GLint success = 0;
GLchar errorLog[1024] = { 0 };
//линкуем программу
glLinkProgram(shaderProgram);
//проверим, что линковка прошла успешно
glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
if (success == 0) {
glGetProgramInfoLog(shaderProgram, sizeof(errorLog), nullptr, errorLog);
std::cerr « "Error linking shader program: '" « errorLog « "'" « std::endl;
exit(1);
}

//валидируем программу
glValidateProgram(shaderProgram);
//проверим, что валидация прошла успешно
glGetProgramiv(shaderProgram, GL_VALIDATE_STATUS, &success);
if (success == 0) {
glGetProgramInfoLog(shaderProgram, sizeof(errorLog), nullptr, errorLog);
std::cerr « "Invalid shader program: '" « errorLog « "'" « std::endl;
exit(1);
}

//укажем OpenGL, что надо использовать эту программу
glUseProgram(shaderProgram);

//сохраним местоположение переменной gWorld
gWorldLocation = glGetUniformLocation(shaderProgram, "gWorld");
assert(gWorldLocation != 0xFFFFFFFF);
}
 ```
## 2. Вращение
В функции отрисовки меняем матрицу World
```C++
World[0][0] = cosf(Scale); World[0][1] = -sinf(Scale); World[0][2] = 0.0f; World[0][3] = 0.0f;
World[1][0] = sinf(Scale); World[1][1] = cosf(Scale); World[1][2] = 0.0f; World[1][3] = 0.0f;
World[2][0] = 0.0f; World[2][1] = 0.0f; World[2][2] = 1.0f; World[2][3] = 0.0f;
World[3][0] = 0.0f; World[3][1] = 0.0f; World[3][2] = 0.0f; World[3][3] = 1.0f;
```
## 3. Преобразования масштаба
 В функции отрисовки меняем матрицу World
```C++
World[0][0]=sinf(Scale); World[0][1]=0.0f; World[0][2]=0.0f; World[0][3]=0.0f;
World[1][0] = 0.0f; World[1][1] = cosf(Scale); World[1][2] = 0.0f; World[1][3] = 0.0f;
World[2][0] = 0.0f; World[2][1] = 0.0f; World[2][2] = sinf(Scale); World[2][3] = 0.0f;
World[3][0] = 0.0f; World[3][1] = 0.0f; World[3][2] = 0.0f; World[3][3] = 1.0f;
```
## 4. Объединение преобразований
Создадим класс, необходимый для преобразований
```C++
class Pipeline
{
public:
Pipeline()
{
m_scale = glm::vec3(1.0f, 1.0f, 1.0f);
m_worldPos = glm::vec3(0.0f, 0.0f, 0.0f);
m_rotateInfo = glm::vec3(0.0f, 0.0f, 0.0f);
}

//функции задания изменения масштаба
void Scale(float ScaleX, float ScaleY, float ScaleZ)
{
m_scale.x = ScaleX;
m_scale.y = ScaleY;
m_scale.z = ScaleZ;
}

//функция задания изменения положения
void WorldPos(float x, float y, float z)
{
m_worldPos.x = x;
m_worldPos.y = y;
m_worldPos.z = z;
}

//функция задания вращения
void Rotate(float RotateX, float RotateY, float RotateZ)
{
m_rotateInfo.x = RotateX;
m_rotateInfo.y = RotateY;
m_rotateInfo.z = RotateZ;
}

//функция получения итоговой матрицы
const glm::mat4* getTransformation()
private:

//вспомогательные функции
void InitScaleTransform(glm::mat4& m) const;
void InitRotateTransform(glm::mat4& m) const;
void InitTranslationTransform(glm::mat4& m) const;

//необходимые переменные
glm::vec3 mScale;
glm::vec3 mWorldPos;
glm::vec3 mRotateInfo;
glm::mat4 mTransformation;
};
```
В функции отрисовки создадим объект этого класса
```C++
Pipeline p;
p.Scale(sinf(Scale * 0.1f), sinf(Scale * 0.1f), sinf(Scale * 0.1f));
p.WorldPos(sinf(Scale), 0.0f, 0.0f);
p.Rotate(sinf(Scale) * 90.0f, sinf(Scale) * 90.0f, sinf(Scale) * 90.0f);
glUniformMatrix4fv(gWorldLocation, 1, GL_TRUE, (const GLfloat*)p.getTransformation());
```
## 5. Проекция перспективы 
В класс Pipeline добавим метод и необходимые переменые
```C++
struct {
float FOV;
float Width;
float Height;
float zNear;
float zFar;
} m_persProj;

void SetPerspectiveProj(float FOV, float Width, float Height, float zNear, float zFar)
{
m_persProj.FOV = FOV;
m_persProj.Width = Width;
m_persProj.Height = Height;
m_persProj.zNear = zNear;
m_persProj.zFar = zFar;
}
```
В метод getTransformation добавим в умножение матрицу перспективы
```C++
mTransformation = persProjTrans * translationTrans * rotateTrans * scaleTrans;
```
В шейдер вершин добавим вычисление цвета
```C++
Color = vec4(clamp(Position, 0.0, 0.5), 1.0);  
```
Создадим шейдер фрагментов для установки цвета
```C++
#version 330                                                           
in vec4 Color;
out vec4 FragColor;
void main() {
    // Устанавливаем цвет                                                    
    FragColor = Color;
} 
```
Изменим преобразования матрицы
```C++
Pipeline p;
//меняем масштаб
p.Scale(0.1f, 0.1f, 0.1f);
//вращаем фигуру
p.Rotate(0, Scale, 0);
//устанавливаем положение фигуры
p.WorldPos(0.0f, 0.0f, 100.0f);
//задаём проекцию перспективы
p.SetPerspectiveProj(90.0f, WINDOW_WIDTH, WINDOW_HEIGHT, 10.0f, 10000.0f);
```
