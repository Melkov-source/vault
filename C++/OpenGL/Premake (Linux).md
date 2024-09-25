Подключение библиотек:
```lua
filter "system:linux"
	links
	{
		"GL",         -- OpenGL
		"GLU",        -- OpenGL Utility Library
		"GLEW",       -- OpenGL Extension Wrangler Library
		"glfw",       -- GLFW Library for creating windows and context
		"X11",        -- X Window System libraries
		"pthread",    -- POSIX threads
		"Xrandr",     -- X Resize and Rotate
		"Xi",         -- X Input extension
		"dl",         -- Dynamic linking library
		"glut",       -- OpenGL Utility Toolkit
	}
```

Установка GLUT:
```bash
sudo apt-get install freeglut3 freeglut3-dev
```

```bash
ls /usr/lib/x86_64-linux-gnu | grep glut
```

