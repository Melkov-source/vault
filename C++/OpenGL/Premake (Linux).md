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
		"GLUT"        -- OpenGL Utility Toolkit
	}
```

