# **4coder CMake Builder**

### Purpose:

Perform super builds of custom layer using cmake, making it less operating system dependent, but simple single cmake script, which also generates a compile_commands.json which is helpful to those who are more familiar with LSP supporting editors, making it easier to parse through 4coder code base, and enjoy, with simple transition into 4coder eventually.

### Usage:

1. Clone the repo into the root of your 4coder directory.

   ```bash
   git clone https://github.com/Borwe/4coder_cmake_builder.git
   ```

2. Run the cmake command to setup build directory for building.

   ```bash
   cmake -Bbuild ./4coder_cmake_builder
   ```

3. Go into the build directory and build the project.

   ```bash
   cd build && make
   ```

4. You should now have a compile_commands.json on your root 4coder directory, and an update custom layer library there too.



### Contributions:

All contributions are welcome. Post in issue section for any issues.

