+++
title = "kotlin arrow option相互変換チートシート"
date = 2020-09-19
[taxonomies]
tags = ["kotlin", "arrow"]
+++

kotlinで[arrow](https://arrow-kt.io/)を使ったときにarrowのoptionとkotlinのnullableとjavaのoptionalを相互にどうやって変換すればよいかをまとめた

<!-- more -->

## arrowとは
[arrow](https://arrow-kt.io/)とはkotlinに関数型プログラミングの機能を追加するライブラリ。
## コード
`ディレクトリ構成`
```
.
├── build.gradle
└── src
    └── main
        ├── java
        │   └── kotlinsandbox
        │       └── MyJavaClass.java
        └── kotlin
            └── kotlinsandbox
                ├── App.kt
                └── MyKotlinClass.kt

```

`build.gradle`
```gradle
plugins {
    id 'org.jetbrains.kotlin.jvm' version '1.4.0'
    id 'application'
}
apply plugin: 'kotlin-kapt'

repositories {
    mavenCentral()
    maven { url "https://dl.bintray.com/arrow-kt/arrow-kt/" }
}

def arrow_version = "0.10.5"
dependencies {
    implementation platform('org.jetbrains.kotlin:kotlin-bom')
    implementation 'org.jetbrains.kotlin:kotlin-stdlib-jdk8'

    implementation "io.arrow-kt:arrow-core:$arrow_version"
    implementation "io.arrow-kt:arrow-syntax:$arrow_version"
    kapt    "io.arrow-kt:arrow-meta:$arrow_version"
}

application {
    mainClassName = 'kotlinsandbox.AppKt'
}
```

`App.kt`
```kotlin
package kotlinsandbox

import arrow.core.Option
import arrow.core.none
import arrow.core.toOption
import java.util.*


fun main(args: Array<String>) {
    // java(platform type) -> kotlin nullable
    var nullable: String? = MyJavaClass.platformString()
    // java(platform type) -> kotlin arrow option
    var option1: Option<String> = Option.fromNullable(MyJavaClass.platformString())
    var option2: Option<String> = MyJavaClass.platformString().toOption()
    // java(platform type) -> java optional
    var optional: Optional<String> = Optional.ofNullable(MyJavaClass.platformString())

    // java optional -> kotlin nullable
    nullable = MyJavaClass.optionalString().orElse(null)
    // java optional -> kotlin arrow option
    option1 = Option.fromNullable(MyJavaClass.optionalString().orElse(null))
    option2 = MyJavaClass.optionalString().orElse(null).toOption()

    val nullString: String? = null
    // kotlin nullable -> kotlin arrow option
    option1 = Option.fromNullable(nullString)
    option2 = nullString.toOption()
    // kotlin nullable -> java optional
    optional = Optional.ofNullable(nullString)

    val optionString: Option<String> = none()
    // kotlin arrow option -> kotlin nullable
    nullable = optionString.orNull()
    // kotlin arrow option -> java optional
    optional = Optional.ofNullable(optionString.orNull())
}
```

`MyJavaClass.java`
```java
package kotlinsandbox;

import java.util.Optional;

public class MyJavaClass {
    public static String platformString() {
        return null;
    }

    public static Optional<String> optionalString() {
        return Optional.empty();
    }

    // javaでの変換の方法も書いておく
    public static void convert() {
        // java(platform type) -> java optional
        Optional<String> optional = Optional.ofNullable(platformString());
        // kotlin nullable -> java optional
        optional = Optional.ofNullable(MyKotlinClass.nullableString());
        // kotlin arrow option -> java optional
        optional = Optional.ofNullable(MyKotlinClass.optionString().orNull());
    }
}
```

`MyKotlinClass.kt`
```kotlin
package kotlinsandbox

import arrow.core.Option
import arrow.core.none

class MyKotlinClass {
    companion object {
        @JvmStatic
        fun nullableString(): String? {
            return null
        }

        @JvmStatic
        fun optionString(): Option<String> {
           return none()
        }
    }
}
```

