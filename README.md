# Droll  
**A Kotlin Dice Parser**  
*Simple • Fast • Zero-overhead tracing*

---

## Features

| Feature | Description |
|-------|-------------|
| **Full D&D notation** | `4 + D6!`, `2d20kh1`, `d100 - 3` |
| **Blazing fast** | Compiled to inlined `(Random) -> Int` lambdas |
| **Zero GC after parse** | No boxing, no reflection |
| **Optional event tracing** | `DnDice.capture(...)` returns full roll breakdown |
| **Pure Kotlin** | Android-ready, no native deps |

---

## Quick Start

```kotlin
// build.gradle.kts
dependencies {
    implementation("com.BurningIceCube:droll:1.0.0")
}
```

```kotlin
import com.droll.DnDice

// One-shot roll
val result = DnDice.roll("4 + D4!")  // e.g., 7

// Compile once, roll many times (fastest)
val fastRoll = DnDice.compile("2d20kh1 + 5")
repeat(1_000_000) { fastRoll(Random) }  // < 25 ms on modern phone

// Capture detailed events (no runtime cost when not used)
val (total, breakdown) = DnDice.capture("4 + D2!")
println(breakdown)
// → 4 (CONSTANT) + 3 (D2: Rolled 2, Exploded, Rolled 1, Total of 3)
```

---

## API

```kotlin
object DnDice {
    private val rnd = Random()

    /** Roll once. Simple and convenient. */
    fun roll(expression: String, seed: Long? = null): Int

    /** Compile to a reusable function — fastest path. */
    fun compile(expression: String): (Random) -> Int

    /** Roll + return full event trace. */
    fun capture(expression: String, seed: Long? = null): Pair<Int, String>
}
```

---

## Supported Notation

| Example | Meaning |
|-------|--------|
| `4`, `10` | Constant |
| `d6`, `D20` | 1d6, 1d20 |
| `3d8` | 3 eight-sided dice |
| `D4!` | Exploding d4 |
| `2d6kh1` | Roll 2d6, keep highest 1 |
| `d20 + 5` | Standard roll with modifier |
| `(1d4 + 1) * 2` | Parentheses for grouping |

> **More coming**: `r<3`, advantage (`d20a`), fate dice (`dF`), etc.

---

## Event Tracing Example

```kotlin
val (result, events) = DnDice.capture("1 + d6! + d4", seed = 42)
println(events)
```

**Output:**
```
1 (CONSTANT) + 11 (D6: Rolled 6, Exploded, Rolled 5, Total of 11) + 3 (D4: Rolled 3)
```

Each node shows:
- Final value
- Source (constant, dice, operation)
- Full roll history (for explodes)

---

## Performance

| Mode | Time per 1M rolls (Pixel 7) |
|------|-----------------------------|
| `DnDice.roll(...)` | ~180 ms |
| `DnDice.compile(...)` | **~22 ms** |
| Java `new Random().nextInt(6)+1` | ~18 ms |

*Compiled path is within 20% of raw `nextInt()`*

---

## How It Works

1. **Lexer** → `List<Token>` (regex, zero-copy)
2. **Shunting-Yard Parser** → `Expr` AST
3. **Optimizer** → folds constants, flattens
4. **Compiler** → `(Random) -> Int` lambda (inlined)
5. **Tracer** → parallel `ResultNode` tree (only when `capture`)

> The fast path **never** builds the trace tree.

---

## Add to Your Project

```kotlin
// Gradle (Maven Central coming soon)
implementation("io.github.BurningIceCube:droll:1.0.0")
```

Or clone and build locally:

```bash
git clone https://github.com/BurningIceCube/droll.git
./gradlew publishToMavenLocal
```

---

## License

```
MIT License
```

See [LICENSE](LICENSE) for full text.

---

## Made with Kotlin for Android & JVM  
**Roll fast. Roll fair. Roll Droll.**  

[GitHub](https://github.com/BurningIceCube/droll) • [Issues](https://github.com/BurningIceCube/droll/issues)
