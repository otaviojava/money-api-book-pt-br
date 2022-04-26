## Trabalhando com Streams

Uma vez que a API nasceu após o Java 8, é natural que ele tenha suporte a alguns recursos do Java 8, como já vimos no capítulo anterior, a busca de uma cotação é data a partir de um ```LocalDate```, sem falar no suporte a nova estrutura de dados que melhorou a experiência em trabalhar com listas, o **Stream**. Trabalhar com coleções é muito importante e comum, por exemplo, uma lista de produtos que serão cobrados é natural que seja impresso o valor total dessa compra. Para trabalhar com Stream a referência de implementação tem a classe ```MonetaryFunctions```.

### Ordenando uma lista monetária


Dentro da classe `MonetaryFunctions` é possível ordenar pela moeda, pelo valor numérico apenas, além da valiosidade de um dinheiro, levando em consideração a cotação da moeda, de forma ascendente e descendente. 

#### Realizando ordenação com moeda

No caso da ordenação da moeda é levado em consideração o código da moeda. Por exemplo, uma lista com as moedas `USD`, `EUR`, `BRL` retornará `BRL`, `EUR` e `USD` de forma ascendente e USD, EUR, BRL de forma decrescente.

```java
public class SortMonetaryAmountCurrency {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit euro = Monetary.getCurrency("EUR");
        CurrencyUnit real = Monetary.getCurrency("BRL");

        MonetaryAmount money = Money.of(9, euro);
        MonetaryAmount money2 = Money.of(10, dollar);
        MonetaryAmount money3 = Money.of(11, real);

        List<MonetaryAmount> resultAsc = Stream.of(money, money2, money3)
                .sorted(MonetaryFunctions
                        .sortCurrencyUnit()).collect(Collectors.toList());//[BRL 11, EUR 9, USD 10]
        List<MonetaryAmount> resultDesc = Stream.of(money, money2, money3)
                .sorted(MonetaryFunctions
                        .sortCurrencyUnitDesc()).collect(Collectors.toList());//[USD 10, EUR 9, BRL 11]

    }
}
```

#### Realizando ordenação com o valor numérico

A ordenação pelo valor numérico ignora a moeda e ordena apenas levando em consideração o valor monetário, vale salientar, que essa ordenação não realiza cotação de valores, em outras palavras, o valor de dez reais terá o mesmo valor que dez dólares. Também é possível retornar de forma ascendente e descendente.

```java
public class SortMonetaryAmountNumber {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit euro = Monetary.getCurrency("EUR");
        CurrencyUnit real = Monetary.getCurrency("BRL");

        MonetaryAmount money = Money.of(9, euro);
        MonetaryAmount money2 = Money.of(10, dollar);
        MonetaryAmount money3 = Money.of(11, real);

        List<MonetaryAmount> resultAsc = Stream.of(money, money2, money3)
                .sorted(MonetaryFunctions
                        .sortNumber()).collect(Collectors.toList());//[EUR 9, USD 10, BRL 11]
        List<MonetaryAmount> resultDesc = Stream.of(money, money2, money3)
                .sorted(MonetaryFunctions
                        .sortNumberDesc()).collect(Collectors.toList());//[BRL 11, USD 10, EUR 9]
    }
}
```

#### Realizando ordenação levando em consideração a cotação


Também é possível realizar uma ordenação de forma crescente e decrescente levanto em consideração a cotação da moeda. Para isso basta passar uma implementação de `ExchangeRateProvider`. Por exemplo, dado uma lista com dez dólares, onze reais e nove euros, retornará de forma ascendente o valor de onze reais, dez dólares e nove euros levando em consideração que pela cotação o dólar é mais valioso que o real e menos que valioso que o euro.


```java
public class SortMonetaryAmountExchange {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit euro = Monetary.getCurrency("EUR");
        CurrencyUnit real = Monetary.getCurrency("BRL");

        MonetaryAmount money = Money.of(9, euro);
        MonetaryAmount money2 = Money.of(10, dollar);
        MonetaryAmount money3 = Money.of(11, real);

        ExchangeRateProvider provider =
                MonetaryConversions.getExchangeRateProvider(ExchangeRateType.IMF);

    
        List<MonetaryAmount> resultAsc = Stream.of(money, money2, money3)
                .sorted(MonetaryFunctions
                        .sortValuable(provider))
                .collect(Collectors.toList());//[BRL 11, EUR 9, USD 10]

        List<MonetaryAmount> resultDesc = Stream.of(money, money2, money3)
                .sorted(MonetaryFunctions
                        .sortValuableDesc(provider)).collect(Collectors.toList());//[USD 10, EUR 9, BRL 11]

    }
}
```

#### Juntando as ordenações



Apenas como recordação, já que esse recurso não é da money-api e sim do Java 8, é possível também misturar mais de um ordenador, para isso basta utilizar o método **thenComparing**. Basicamente ele faz a ordenação e caso os valores tenham o mesmo peso, ao usar o compare retorne o valor zero, ele usará o outro ordenador, assim a ordem que for definida o sort influenciará no resultado da ordenação.


```java
public class SortMixMonetaryAmountNumberCurrency {

    public static void main(String[] args) {
        CurrencyUnit dollar = Monetary.getCurrency("USD");
        CurrencyUnit euro = Monetary.getCurrency("EUR");
        CurrencyUnit real = Monetary.getCurrency("BRL");

        MonetaryAmount money = Money.of(10, euro);
        MonetaryAmount money2 = Money.of(10, dollar);
        MonetaryAmount money3 = Money.of(10, real);
        MonetaryAmount money4 = Money.of(9, real);
        MonetaryAmount money5 = Money.of(8, dollar);

        List<MonetaryAmount> resultAsc = Stream.of(money, money2, money3, money4, money5)
                .sorted(MonetaryFunctions
                        .sortNumber().thenComparing(MonetaryFunctions.sortCurrencyUnit()))
                .collect(Collectors.toList());//[USD 8, BRL 9, BRL 10, EUR 10, USD 10]
        List<MonetaryAmount> resultDesc = Stream.of(money, money2, money3, money4, money5)
                .sorted(MonetaryFunctions
                        .sortNumberDesc().thenComparing(MonetaryFunctions.sortCurrencyUnitDesc()))
                .collect(Collectors.toList());//[USD 10, EUR 10, BRL 10, BRL 9, USD 8]
        //using currency first
        List<MonetaryAmount> resultCurrencyAsc = Stream.of(money, money2, money3, money4, money5)
                .sorted(MonetaryFunctions
                        .sortCurrencyUnit().thenComparing(MonetaryFunctions.sortNumber()))
                .collect(Collectors.toList());//[BRL 9, BRL 10, EUR 10, USD 8, USD 10]
        List<MonetaryAmount> resultCurrencyDesc = Stream.of(money, money2, money3, money4, money5)
                .sorted(MonetaryFunctions
                        .sortCurrencyUnitDesc().thenComparing(MonetaryFunctions.sortNumberDesc()))
                .collect(Collectors.toList());//[USD 10, USD 8, EUR 10, BRL 10, BRL 9]

    }
}
```

