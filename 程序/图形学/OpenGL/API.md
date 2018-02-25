# 顶点缓冲
* 创建一个顶点缓冲
	<pre><code>
    GLuint VBO; 
	glGenBuffers(1, &VBO); 
	glGenBuffers 的第一个参数表示生成的缓冲数量，比如创建 5 个 VBO
	GLuint VBOs[5];
	glGenBuffers(5, VBOs);
    </code></pre>
* 缓冲有很多类型，下面的语句把新建的缓冲设置为当前的 GL_ARRAY_BUFFER 类型缓冲
    <pre><code>glBindBuffer(GL_ARRAY_BUFFER, VBO); </code></pre>
* 把顶点写入当前缓冲
	<pre><code>glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);</code></pre>
	第4个参数可选值如下，
	* GL_STATIC_DRAW ：数据不会或几乎不会改变。
	* GL_DYNAMIC_DRAW：数据会被改变很多。
	* GL_STREAM_DRAW ：数据每次绘制时都会改变。显卡会把动态的数据放到高速写入的内存部分。

# Shader编译
* 创建顶点着色器
    <pre><code>
    GLuint vertexShader;
	vertexShader = glCreateShader(GL_VERTEX_SHADER);
    </code></pre>
	得到一个用 ID 表示的着色器对象 vertexShader
* 编译顶点着色器
	* 创建着色器并编译
		<pre><code>
        glShaderSource(vertexShader, 1, &vertexShaderSource, NULL); 
		glCompileShader(vertexShader);
        </code></pre>
		glShaderSource 的第一个参数是着色器对象。第二个参数是传递的源码数组大小。第三个是源码字符串数组，注意是源码数组，类型为 char**。第四个是指定每个数组大小的数组。
	* 检测是否编译成功
		<pre><code>
        GLint success;
        GLchar infoLog[512];
        glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
        </code></pre>
		glGetShaderiv 可以获取当前 shader 的状态，第二个参数是想要获取的状态类型，可以是GL_SHADER_TYPE, GL_DELETE_STATUS, GL_COMPILE_STATUS, GL_INFO_LOG_LENGTH, GL_SHADER_SOURCE_LENGTH。这里我们想要获取上一次的编译状态，所以是GL_COMPILE_STATUS。
		○ 如果编译失败，可以用 glGetShaderInfoLog获取错误信息
        <pre><code>
		if(!success)
        {
            glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
            std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
        }
        </code></pre>
	* 编译片段着色器
	    与编译顶点着色器类似，只是创建的时候把类型换成GL_FRAGMENT_SHADER。
	    <pre><code>
        GLuint fragmentShader; 
        fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
        glShaderSource(fragmentShader, 1, &fragmentShaderSource, null); 
        glCompileShader(fragmentShader);
        </code></pre>
	* 链接着色器程序
		* 创建着色器程序与链接
            <pre><code>
            GLuint shaderProgram; 
            shaderProgram = glCreateProgram();
            glAttachShader(shaderProgram, vertexShader); 
            glAttachShader(shaderProgram, fragmentShader); 
            glLinkProgram(shaderProgram);
            </code></pre>
		* 检测是否成功
		    <pre><code>
            glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
            if(!success) {
                glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
                ...
            }
            </code><pre>
		* 使用着色器程序
		    <pre><code>glUseProgram(shaderProgram);</code></pre>
		* 对象清理
		    链接成功后，就不需要着色器对象了，需要删除他们
		    <pre><code>
            glDeleteShader(vertexShader); 
		    glDeleteShader(fragmentShader);
            </code></pre>

# 链接顶点属性
顶点缓冲中的顶点由3个浮点数组成，顶点的3个值之间没有空隙。

用glVertexAttribPointer函数告诉OpenGL该如何解析顶点数据
<pre><code>
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*)0);
glEnableVertexAttribArray(0);</pre></code>

* 第一个参数表示顶点数据的索引，在顶点着色器可以通过这个索引来访问这组顶点数据。
* 第二个参数表示顶点属性的大小，有效值是1,2,3,4
* 第三个参数表示顶点属性的类型
* 第四个参数表示表示是否要把顶点数据标准化（normalize）
* 第五个参数叫做步长，表示连续的顶点属性组之间的距离。
* 第六个参数表示顶点数据的起始偏移量，是个 GLvoid* 类型的值

顶点数组默认是禁用的，所以要用 glEnableVertexAttribArray来启用。

显示一个物体的完整代码如下

<pre><code>
// 0. 复制顶点数组到缓冲中供OpenGL使用 
glBindBuffer(GL_ARRAY_BUFFER, VBO); 
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
// 1. 设置顶点属性指针 
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*)0); 
glEnableVertexAttribArray(0); 
// 2. 当我们渲染一个物体时要使用着色器程序 
glUseProgram(shaderProgram); 
// 3. 绘制物体 
someOpenGLFunctionThatDrawsOurTriangle(); 
</code></pre>
绘制每个物体都必须重复这个过程，用顶点数组对象可以把所有状态储存在一个对象中，通过绑定这个对象就可以恢复状态。

# 顶点数组对象
顶点数组对象(Vertex Array Object, VAO)可以像顶点缓冲对象那样被绑定，任何随后的顶点属性调用都会储存在这个VAO中。这使在不同顶点数据和属性配置之间切换变得非常简单，只需要绑定不同的VAO就行了。

一个顶点数组对象会储存以下这些内容：
* glEnableVertexAttribArray和glDisableVertexAttribArray的调用。
* 通过glVertexAttribPointer设置的顶点属性配置。
* 通过glVertexAttribPointer调用进行的顶点缓冲对象与顶点属性链接。

VAO只会记录状态的改变，这些都是状态改变函数，如果是生成顶点数组之类不会对状态产生改变的函数，是不会进行存储的。创建一个VAO和创建一个VBO类似：
<pre><code>
GLuint VAO;
glGenVertexArrays(1, &VAO);  
</code></pre>
绑定的代码如下
<pre><code>
// ..:: 初始化代码（只运行一次 (除非你的物体频繁改变)） :: .. 
// 1. 绑定VAO 
glBindVertexArray(VAO); 
// 2. 把顶点数组复制到缓冲中供OpenGL使用 
glBindBuffer(GL_ARRAY_BUFFER, VBO); 
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW); 
// 3. 设置顶点属性指针 
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*)0); 
glEnableVertexAttribArray(0); 
//4. 解绑VAO glBindVertexArray(0); 
[...] 
// ..:: 绘制代（游戏循环中） :: .. 
// 5. 绘制物体 
glUseProgram(shaderProgram); 
glBindVertexArray(VAO); 
someOpenGLFunctionThatDrawsOurTriangle(); 
glBindVertexArray(0);
</code></pre>
# 绘图API
指定渲染的图元
<pre><code>
glDrawArrays(GL_TRIANGLES, 0, 3);
</code></pre>
* 参数一表示图元类型，可以接受的类型有GL_POINTS, GL_LINE_STRIP, GL_LINE_LOOP, GL_LINES, GL_TRIANGLE_STRIP, GL_TRIANGLE_FAN,  GL_TRIANGLES
* 参数二表示第一个元素的索引
* 参数三表示需要渲染的顶点索引数目

# 索引缓冲对象
索引缓冲对象(Element Buffer Object，EBO，也叫Index Buffer Object，IBO)。如果不用索引，那么渲染多个三角形的时候，就可能存在重复的顶点。比如绘制一个矩形，不用索引就需要6个点，用索引只要4个点。使用方法如下，
<pre><code>
GLuint EBO; 
glGenBuffers(1, &EBO);
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO); 
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW); 
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
</code></pre>
注意如果用索引绘制需要用glDrawElements来代替glDrawArrays。