# Задача

- Мы даём вам ужасно написанный, но типа работающий код
- Задача: отрефакторить (фактически переписать заново) этот код правильно


# Код

```php
<?php

foreach (explode("\n", file_get_contents($argv[1])) as $row) {

    if (empty($row)) break;
    $p = explode(",",$row);
    $p2 = explode(':', $p[0]);
    $value[0] = trim($p2[1], '"');
    $p2 = explode(':', $p[1]);
    $value[1] = trim($p2[1], '"');
    $p2 = explode(':', $p[2]);
    $value[2] = trim($p2[1], '"}');

    $binResults = file_get_contents('https://lookup.binlist.net/' .$value[0]);
    if (!$binResults)
        die('error!');
    $r = json_decode($binResults);
    $isEu = isEu($r->country->alpha2);

    $rate = @json_decode(file_get_contents('http://api.exchangeratesapi.io/v1/latest?access_key=08914569ac738cfaad9d780d06135caf'), true)['rates'][$value[2]];
    if ($value[2] == 'EUR' or $rate == 0) {
        $amntFixed = $value[1];
    }
    if ($value[2] != 'EUR' or $rate > 0) {
        $amntFixed = $value[1] / $rate;
    }

    echo $amntFixed * ($isEu == 'yes' ? 0.01 : 0.02);
    print "\n";
}

function isEu($c) {
    $result = false;
    switch($c) {
        case 'AT':
        case 'BE':
        case 'BG':
        case 'CY':
        case 'CZ':
        case 'DE':
        case 'DK':
        case 'EE':
        case 'ES':
        case 'FI':
        case 'FR':
        case 'GR':
        case 'HR':
        case 'HU':
        case 'IE':
        case 'IT':
        case 'LT':
        case 'LU':
        case 'LV':
        case 'MT':
        case 'NL':
        case 'PO':
        case 'PT':
        case 'RO':
        case 'SE':
        case 'SI':
        case 'SK':
            $result = 'yes';
            return $result;
        default:
            $result = 'no';
    }
    return $result;
}

```

## Пример файла `input.txt`

```
{"bin":"45717360","amount":"100.00","currency":"EUR"}
{"bin":"516793","amount":"50.00","currency":"USD"}
{"bin":"45417360","amount":"10000.00","currency":"JPY"}
{"bin":"41417360","amount":"130.00","currency":"USD"}
{"bin":"4745030","amount":"2000.00","currency":"GBP"}

```

## Запуск кода

Предположим, php-код находится в файле `app.php`, он будет вызываться такой командой из консоли (выдача может отличаться, так как данные динамичные):
```
> php app.php input.txt
1
0.46180844185832
1.6574127786525
2.4014038976632
43.714413735069

```

# Как работает код

1. Идея кода - высчитывать комиссии для уже совершённых транзакций;
2. Список транзакций находится в файле txt, по одной транзакции на строку, в JSON;
3. BIN-числа - это первые цифры кредитной карты, их можно использовать для определения страны, в которой была выпущена карта;
4. Для стран в составе Евросоюза и за его пределами комиссия разная;
5. Все комиссии считаются в евро (EUR).

# Требование к вашему коду

1. В качестве улучшения добавьте округление до центов, например вместо `0.46180...` должно получиться `0.47`.
1. Во всём остальном ваш код должен выдавать те же значения, что и оригинальный код
1. Код должен быть расширяемым, он должен учитывать, что мы можем:
    1. Сменить провайдера, выдающего информацию о курсах валют
    2. Сменить провайдера, определяющего страну по BIN-числам
    3. На всякий случай: ничего дополнительного не дописываем. Нужно переделать структуру, но не добавлять доп. функционал.
1. Вы можете использовать любые дополнительные библиотеки (если они понадобятся), подключённые через composer
