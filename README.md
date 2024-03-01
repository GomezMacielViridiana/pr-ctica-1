#include <stdio.h>
#include <string.h>
#include <glew.h>
#include <glfw3.h>
#include <stdlib.h>
#include <time.h>

const int WIDTH = 800, HEIGHT = 600;
GLuint VAO[3], VBO[3], shader;
// 421530876  Gómez Maciel Viridiana
// Coordenadas de los triángulos para cada letra
GLfloat vertices_V[] = {
    -0.8f, 0.6f, 0.0f,
    -0.4f, -0.6f, 0.0f,
    -0.2f, 0.6f, 0.0f,
    -0.2f, 0.6f, 0.0f,
    0.0f, -0.6f, 0.0f,
    0.2f, 0.6f, 0.0f
};

GLfloat vertices_G[] = {
    0.2f, 0.6f, 0.0f,
    0.4f, 0.6f, 0.0f,
    0.0f, 0.6f, 0.0f,
    0.0f, 0.6f, 0.0f,
    0.4f, -0.6f, 0.0f,
    0.2f, -0.6f, 0.0f
};

GLfloat vertices_M[] = {
    0.8f, 0.6f, 0.0f,
    0.8f, -0.6f, 0.0f,
    0.6f, 0.6f, 0.0f,
    0.6f, 0.6f, 0.0f,
    1.0f, -0.6f, 0.0f,
    1.0f, 0.6f, 0.0f
};

// Vertex Shader
static const char* vShader = "                                     \n\
#version 330                                                        \n\
layout (location = 0) in vec3 pos;                                  \n\
void main() {                                                       \n\
    gl_Position = vec4(pos.x, pos.y, pos.z, 1.0);                   \n\
}";

// Fragment Shader
static const char* fShader = "                                     \n\
#version 330                                                        \n\
out vec4 color;                                                     \n\
void main() {                                                       \n\
    color = vec4(1.0f, 0.0f, 0.0f, 1.0f);                          \n\
}";

void CreateTriangle(GLfloat* vertices, int size, int index) {
    glGenVertexArrays(1, &VAO[index]);
    glBindVertexArray(VAO[index]);

    glGenBuffers(1, &VBO[index]);
    glBindBuffer(GL_ARRAY_BUFFER, VBO[index]);
    glBufferData(GL_ARRAY_BUFFER, size, vertices, GL_STATIC_DRAW);

    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*)0);
    glEnableVertexAttribArray(0);

    glBindBuffer(GL_ARRAY_BUFFER, 0);
    glBindVertexArray(0);
}

void AddShader(GLuint theProgram, const char* shaderCode, GLenum shaderType) {
    GLuint theShader = glCreateShader(shaderType);
    const GLchar* theCode[1];
    theCode[0] = shaderCode;
    GLint codeLength[1];
    codeLength[0] = strlen(shaderCode);
    glShaderSource(theShader, 1, theCode, codeLength);
    glCompileShader(theShader);
    GLint result = 0;
    GLchar eLog[1024] = { 0 };
    glGetShaderiv(theShader, GL_COMPILE_STATUS, &result);
    if (!result) {
        glGetProgramInfoLog(shader, sizeof(eLog), NULL, eLog);
        printf("EL error al compilar el shader %d es: %s \n", shaderType, eLog);
        return;
    }
    glAttachShader(theProgram, theShader);
}

void CompileShaders() {
    shader = glCreateProgram();
    if (!shader) {
        printf("Error creando el shader");
        return;
    }
    AddShader(shader, vShader, GL_VERTEX_SHADER);
    AddShader(shader, fShader, GL_FRAGMENT_SHADER);
    GLint result = 0;
    GLchar eLog[1024] = { 0 };
    glLinkProgram(shader);
    glGetProgramiv(shader, GL_LINK_STATUS, &result);
    if (!result) {
        glGetProgramInfoLog(shader, sizeof(eLog), NULL, eLog);
        printf("EL error al linkear es: %s \n", eLog);
        return;
    }
    glValidateProgram(shader);
    glGetProgramiv(shader, GL_VALIDATE_STATUS, &result);
    if (!result) {
        glGetProgramInfoLog(shader, sizeof(eLog), NULL, eLog);
        printf("EL error al validar es: %s \n", eLog);
        return;
    }
}

// Función para generar un valor aleatorio en el rango [min, max]
float randomFloat(float min, float max) {
    return min + static_cast <float> (rand()) / (static_cast <float> (RAND_MAX / (max - min)));
}

int main()
{
    if (!glfwInit()) {
        printf("Falló inicializar GLFW");
        glfwTerminate();
        return 1;
    }

    // Asignando variables de GLFW y propiedades de ventana
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
    glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);

    // CREAR VENTANA
    GLFWwindow* mainWindow = glfwCreateWindow(WIDTH, HEIGHT, "Primer ventana", NULL, NULL);
    if (!mainWindow) {
        printf("Fallo en crearse la ventana con GLFW");
        glfwTerminate();
        return 1;
    }
    int BufferWidth, BufferHeight;
    glfwGetFramebufferSize(mainWindow, &BufferWidth, &BufferHeight);
    glfwMakeContextCurrent(mainWindow);
    glewExperimental = GL_TRUE;
    if (glewInit() != GLEW_OK) {
        printf("Falló inicialización de GLEW");
        glfwDestroyWindow(mainWindow);
        glfwTerminate();
        return 1;
    }
    glViewport(0, 0, BufferWidth, BufferHeight);

    CreateTriangle(vertices_V, sizeof(vertices_V), 0);
    CreateTriangle(vertices_G, sizeof(vertices_G), 1);
    CreateTriangle(vertices_M, sizeof(vertices_M), 2);
    CompileShaders();

    // Establecer la semilla para la función rand()
    srand(time(NULL));

    // Loop mientras no se cierra la ventana
    while (!glfwWindowShouldClose(mainWindow))
    {
        // Recibir eventos del usuario
        glfwPollEvents();

        // Limpiar la ventana con un color aleatorio
        glClearColor(randomFloat(0.0f, 1.0f), randomFloat(0.0f, 1.0f), randomFloat(0.0f, 1.0f), 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        glUseProgram(shader);

        glBindVertexArray(VAO[0]);
        glDrawArrays(GL_TRIANGLES, 0, 6);
        glBindVertexArray(0);

        glBindVertexArray(VAO[1]);
        glDrawArrays(GL_TRIANGLES, 0, 6);
        glBindVertexArray(0);

        glBindVertexArray(VAO[2]);
        glDrawArrays(GL_TRIANGLES, 0, 6);
        glBindVertexArray(0);

        glUseProgram(0);

        // Intercambiar los buffers
        glfwSwapBuffers(mainWindow);
    }

    // Terminar GLFW
    glfwTerminate();

    return 0;
}
