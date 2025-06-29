---

title: Gradle 基础
created: 2025-06-28
tags:
    - Gradle
    - Tutorial

---

## Wrapper

不要手动修改 `./gradle/wrapper/` 中的文件和 `gradlew` 和 `gradlew.bat`，而是使用 `gradlew` 和 `gradlew.bat` 来执行 Gradle 命令。

比如需要更新项目中的 Gradle 版本，只需要执行 `./gradlew wrapper --gradle-version 8.0` 即可。
