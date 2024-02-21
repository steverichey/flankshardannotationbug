# Flank Shard Annotation Bug

This project exists to highlight an issue in [Flank](https://github.com/Flank/flank) [v23.10.1](https://github.com/Flank/flank/tree/v23.10.1), namely that annotations specified in test-targets do not respect parameterized tests. Aside from the additional test classes, this is an unchanged default Android Studio project.

This project includes four test classes, each containing a single test:

- [InstrumentedTest](./app/src/androidTest/java/flank/shard/annotationbug/InstrumentedTest.kt)
- [InstrumentedAnnotatedTest](./app/src/androidTest/java/flank/shard/annotationbug/InstrumentedAnnotatedTest.kt)
- [ParameterizedTest](./app/src/androidTest/java/flank/shard/annotationbug/ParameterizedTest.kt)
- [ParameterizedAnnotatedTest](./app/src/androidTest/java/flank/shard/annotationbug/ParameterizedAnnotatedTest.kt)

`Instrumented` test classes are annotated with `@RunWith(AndroidJUnit4::class)`, while `Parameterized` test classes are annotated with `@RunWith(Parameterized::class)`. Methods in the `Annotated` classes are marked with the [@Annotation](./app/src/androidTest/java/flank/shard/annotationbug/Annotation.kt) annotation.

When running the following command with no `test-targets` defined in the [flank.yml](./flank.yml) file:

```bash
java -jar flank.jar android run --dump-shards
```

This results in the following abbreviated [android_shards.json](./android_shards.json) file.

```json
"shard-0": [
    "class flank.shard.annotationbug.InstrumentedTest#useAppContext"
],
"shard-1": [
    "class flank.shard.annotationbug.InstrumentedAnnotatedTest#useAppContext"
],
"shard-2": [
    "class flank.shard.annotationbug.ParameterizedTest"
],
"shard-3": [
    "class flank.shard.annotationbug.ParameterizedAnnotatedTest"
]
```

If we now add the following lines to the `gcloud` section of the `flank.yml` file:

```yaml
test-targets:
    - annotation flank.shard.annotationbug.Annotation
```

We will then get the following abbreviated `android_shards.json` file:

```json
"shard-0": [
    "class flank.shard.annotationbug.InstrumentedAnnotatedTest#useAppContext"
]
```

However, there are _two_ tests that should be included; both the `InstrumentedAnnotatedTest` and the `ParameterizedAnnotatedTest` should be included in the test shards.
