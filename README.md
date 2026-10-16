# ➗ Cryo-Math

Biblioteca de cálculos matemáticos para a linguagem **[Cryo](https://github.com/Victor-477/Cryo)**.

Ela é escrita **100% em Cryo**: não usa nenhum builtin matemático (`sqrt`, `pow`, `floor`...). Seno, logaritmo e exponencial são calculados por séries com redução de intervalo, e a raiz quadrada pelo método de Newton. O resultado é o mesmo em qualquer backend do compilador Burnout. A suíte de testes passa nos backends `pyro`, `node` e `go`.

## Instalação

A Cryo-Math é um pacote Cryo (`cryo.toml`, nome `cryo_math`). No projeto que vai usá-la, declare a dependência pelo caminho:

```toml
# cryo.toml do seu projeto
[package]
name = "meu_app"
version = "0.1.0"

[dependencies]
cryo_math = "../Cryo-Math"
```

E importe com um namespace:

```cryo
import "@cryo_math/math.cryo" as m;

print(m::sqrt(2.0));            // 1.4142135623730951
print(m::sin(m::PI / 6.0));     // 0.5
print(m::factorial(10));        // 3628800
```

Também dá para importar por caminho relativo, sem `cryo.toml`:

```cryo
import "../Cryo-Math/math.cryo" as m;
```

> Ler o `cryo.toml` exige Python 3.11+ (usa o `tomllib` da biblioteca padrão).

## API

Ângulos são sempre em **radianos**. Funções de inteiros recebem e devolvem `int`; o resto usa `number`.

### Constantes

| Nome | Valor |
|---|---|
| `PI`, `TAU`, `HALF_PI` | π, 2π, π/2 |
| `E` | número de Euler |
| `PHI` | razão áurea |
| `SQRT2`, `LN2`, `LN10` | √2, ln 2, ln 10 |
| `EPSILON` | `0.000000001`, tolerância sugerida para `is_close` |

### Básico

| Função | Descrição |
|---|---|
| `abs(x)`, `iabs(n)` | valor absoluto (`number` / `int`) |
| `sign(x)` | `-1`, `0` ou `1` |
| `min(a, b)`, `max(a, b)`, `clamp(x, lo, hi)` | mínimo, máximo, limitar a um intervalo |
| `floor(x)`, `ceil(x)`, `trunc(x)` | arredondamentos para inteiro |
| `round(x)`, `round_to(x, casas)` | arredondamento (.5 se afasta de zero) |
| `frac(x)` | parte fracionária |
| `fmod(a, b)` | resto com o sinal do dividendo (como no C) |
| `mod(a, b)` | resto sempre em `[0, \|b\|)` |
| `is_close(a, b, tol)`, `is_integer(x)` | comparação com tolerância, teste de inteiro |
| `lerp(a, b, t)` | interpolação linear |
| `deg_to_rad(g)`, `rad_to_deg(r)` | conversão de ângulos |

### Potências, raízes e logaritmos

| Função | Descrição |
|---|---|
| `ipow(x, n)` | `x^n` com expoente inteiro (exponenciação por quadrados) |
| `pow(x, y)` | `x^y` com expoente real |
| `square(x)`, `cube(x)` | `x²`, `x³` |
| `sqrt(x)`, `cbrt(x)`, `nth_root(x, n)` | raízes quadrada, cúbica e n-ésima |
| `hypot(a, b)` | `√(a² + b²)` sem overflow intermediário |
| `exp(x)` | `e^x` |
| `ln(x)`, `log10(x)`, `log2(x)`, `log(x, base)` | logaritmos |

### Trigonometria e hiperbólicas

| Função | Descrição |
|---|---|
| `sin`, `cos`, `tan` | seno, cosseno, tangente |
| `asin`, `acos`, `atan`, `atan2(y, x)` | inversas |
| `sinh`, `cosh`, `tanh` | hiperbólicas |
| `asinh`, `acosh`, `atanh` | hiperbólicas inversas |
| `normalize_angle(x)` | reduz um ângulo para `[-π, π]` |

### Inteiros e combinatória

| Função | Descrição |
|---|---|
| `factorial(n)` | `n!` (até `n = 20`, o maior que cabe em `int`) |
| `fib(n)` | n-ésimo Fibonacci (até `n = 92`) |
| `gcd(a, b)`, `lcm(a, b)` | MDC e MMC |
| `is_prime(n)`, `prime_factors(n)` | teste de primalidade, fatoração (`int[]`) |
| `binomial(n, k)`, `permutations(n, k)` | combinação C(n,k) e arranjo A(n,k) |
| `mod_pow(b, e, m)` | `b^e mod m` |
| `is_even(n)`, `is_odd(n)` | paridade |

### Estatística (`number[]`)

| Função | Descrição |
|---|---|
| `sum(xs)` | soma (compensada de Kahan, menos erro de arredondamento) |
| `product(xs)` | produto |
| `mean(xs)`, `median(xs)` | média e mediana |
| `variance(xs)`, `std_dev(xs)` | variância e desvio padrão populacionais |
| `sample_variance(xs)`, `sample_std_dev(xs)` | variância e desvio padrão amostrais (n − 1) |
| `min_of(xs)`, `max_of(xs)` | menor e maior valor |
| `sorted(xs)` | cópia ordenada (o array original não muda) |

### Porcentagem e juros

| Função | Descrição |
|---|---|
| `percent(valor, pct)` | `pct`% de `valor` → `percent(200, 15) = 30` |
| `percent_of(parte, total)` | quanto `parte` representa de `total`, em % |
| `percent_change(antigo, novo)` | variação percentual |
| `compound(capital, taxa, periodos)` | juros compostos (`taxa` 0.05 = 5%) |

## Erros

Argumentos fora do domínio (`sqrt(-1)`, `ln(0)`, `asin(2)`, `fmod(x, 0)`, `factorial(21)`...) lançam uma `string` que começa com `[cryo-math]`. Para tratar, use `try`/`catch`:

```cryo
import "@cryo_math/math.cryo" as m;

fn raiz(number x) -> number ={ return m::sqrt(x); }

try {
    print(raiz(-4.0));
} catch (string e) {
    print(e);    // [cryo-math] sqrt: raiz de numero negativo
}
```

> **Atenção:** na versão atual do compilador, uma chamada `m::funcao(...)` escrita **diretamente** dentro de um bloco `try`, `switch` ou `assert` não é resolvida (erro `unknown function 'm::...'`). Coloque a chamada numa função auxiliar, como `raiz` acima.

## Testes e exemplo

Com o [Burnout](https://github.com/Victor-477/Burnout), o [Cryo](https://github.com/Victor-477/Cryo) (branch `dev`) e o [Pyro](https://github.com/Victor-477/pyro) clonados ao lado desta pasta como `burnout/`, `cryo/` e `pyro/`:

```bash
# suíte de testes (101 verificações; o código de saída é != 0 se alguma falhar)
python burnout/cryoc.py Cryo-Math/tests/test_math.cryo --backend pyro --run

# demonstração
python burnout/cryoc.py Cryo-Math/examples/demo.cryo --backend pyro --run
```

Os testes também rodam com `--backend node` e `--backend go`.

## Licença

MIT. Veja [LICENSE](LICENSE).
