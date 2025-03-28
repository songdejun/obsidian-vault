# CMake
## project()

```cmake
project(<PROJECT-NAME> [<language-name>...])
project(<PROJECT-NAME>
        [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
        [DESCRIPTION <project-description-string>]
        [HOMEPAGE_URL <url-string>]
        [LANGUAGES ...])
```

当这个函数只能在顶层的`CMakeLists.txt`中使用，函数执行后还会设置

- `PROJECT_SOURCE_DIR <PROJECT-NAME>_SOURCE_DIR`项目所在的文件夹的绝对路径。
- `PROJECT_BINARY_DIR <PROJECT-NAME>_BINARY_DIR`项目构建目录（一般是`./build`）的路径。
- `PROJECT_IS_TOP_LEVEL <PROJECT-NAME>_IS_TOP_LEVEL `表明是不是项目顶层`CMakeLists.txt`的布尔值。

可选项指定后还会对应设置变量

- ```cmake
  PROJECT_VERSION <PROJECT-NAME>_VERSION
  PROJECT_VERSION_MAJOR <PROJECT-NAME>_VERSION_MAJOR
  PROJECT_VERSION_MINOR <PROJECT-NAME>_VERSION_MINOR
  PROJECT_VERSION_PATCH <PROJECT-NAME>_VERSION_PATCH
  PROJECT_VERSION_TWEAK  <PROJECT-NAME>_VERSION_TWEAK
  CMAKE_PROJECT_VERSION
  ```
  
- ```cmake
  PROJECT_DESCRIPTION <PROJECT-NAME>_DESCRIPTION
  CMAKE_PROJECT_DESC
  ```

- ```cmake
  PROJECT_HOMEPAGE_URL <PROJECT-NAME>_HOMEPAGE_URL
  CMAKE_PROJECT_HOMEPAGE_URL
  ```

`project()`函数带有代码注入功能。

## add_executable()

```cmake
add_executable(<name> <options>... <sources>...)
```

