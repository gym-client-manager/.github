## FLUTTER TEST

In Flutter si possono eseguire 3 tipi di test:

- Unit Testing: test singoli pezzi di codice, come funzioni o metodi, isolati dal contesto

- Widget Testing: test di widget, reindirizzandoli in un ambiente controllato

- Integration Testing: test dell'interazione tra widget o test dell'interazione tra l'applicazione e i servizi esterni

### UNIT TESTING 

Si usa il `test` package. </br>
Crea i file con il suffisso `_test.dart` e annota le funzioni test con `test`.

Esempio test funzione:

```dart:
int add(int a, int b) => a + b;

void main() {
  test('Add function test', () {
    expect(add(2, 3), 5);
  });
}
```

### WIDGET TESTING

Il test dei widget consiste nel testare la UI della tua applicazione. <br>
Si può usare il package `flutter_test`.

Esempio:

```dart:
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  testWidgets('Widget test example', (WidgetTester tester) async {
    await tester.pumpWidget(MyWidget());
    expect(find.text('Hello, World!'), findsOneWidget);
  });
}
```

### INTEGRATION TESTING

I test di integrazione testano l'interazione tra le parti dell'applicazione. <br> Si può usare il pacchetto `flutter_driver`

Esempio: 

```dart:
void main() {
  group('App Test', () {
    FlutterDriver driver;

    setUpAll(() async {
      driver = await FlutterDriver.connect();
    });

    tearDownAll(() async {
      if (driver != null) {
        driver.close();
      }
    });

    test('Login Test', () async {
      // Implement your login test here
    });
  });
}
```


