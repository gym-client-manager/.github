# RIVERPOD

Riverpod è una framework con cache per la gestione dello stato di un'applicazione 

Utilizzando la programmazione dichiarativa e reattiva, Riverpod è in grado di gestire una grande parte della logica della tua applicazione per te. Può effettuare richieste di rete con gestione degli errori e caching integrati, mentre ri-scarica automaticamente i dati quando necessario.


## IMPOSTARE PROVIDER SCOPE

Prima di iniziare a effettuare richieste di rete, assicurati che `ProviderScope` sia aggiunto nel main dell'applicazione.

Come nell'esempio:

```dart:
void main() {
  runApp(
    // Per installare Riverpod dobbiamo aggiungere questo widget al di   sopra di tutto.
    // Questo widget non dovrebbe essere dentro "MyApp" ma direttamente come parametro di "runApp"
    ProviderScope(
      child: MyApp(),
    ),
  );
}
```
 Con questa dichiarazione di abilita Riverpod per tutta l'applicazione

 ## PRIMA RICHIESTA DI RETE

Eseguire una richiesta di rete è generalmente chiamato "business logic", in Riverpod la business logic è contenuta all'interno dei "provider". 

Un provider è una funzione super potenziata che si comporta come funzione normale, con i vantaggi aggiuntivi di:
- essere cachati
- offrire una gestione degli errori/caricamenti di default
- essere ascoltabili
- eseguirsi automaticamente quando alcuni dati cambiano

Immaginiamo di fare una chiamata ad un'API, ci restituisce un json che convertiremo in un'istanza di una classe. Il passo successivo sarebbe quello di visualizzare questa attività nell'interfaccia utente. Ci assicureremo anche di visualizzare uno stato di caricamento durante la richiesta e di gestire gli errori in modo opportuno

### Definizione del modello

Va definizio un modello dei dati che riceveremo dall'API. Questo modello avrà anche bisogno di un modo per convertire l'oggetto JSON in un'istanza di classe Dart.


In generale è consigliabile utilizzare un code-generator come [Freezed](https://pub.dev/packages/freezed) per gestire la decodifica JSOn. 

Modello esempio: 
```dart:

import 'package:freezed_annotation/freezed_annotation.dart';

part 'activity.freezed.dart';
part 'activity.g.dart';

/// La risposta dell'endpoint `GET /api/activity`.
///
/// È definita utilizzando `freezed` e `json_serializable`..
@freezed
class Activity with _$Activity {
  factory Activity({
    required String key,
    required String activity,
    required String type,
    required int participants,
    required double price,
  }) = _Activity;

  /// Converte un oggetto JSON in un'istanza di [Activity].
  /// Questo consente una lettura type-safe della risposta API.
  factory Activity.fromJson(Map<String, dynamic> json) => _$ActivityFromJson(json);
}

```

### Creazione del provider

La sintassi per definire un provider è come segue:

```dart:
@riverpod
Result myFunction(MyFunctionRef ref) {
  <your logic here>
}
```

L'annotazione:

Tutti i provider devono essere annotati con `@riverpod`- Questa annotazione può essere posizionata su funzioni globali o classi. Mediante quest'annotazione è possibile configurare il provider.


La funzione annotata:

Il nome della funzione annotata determina come verrà interagito con il provider. Data ua funzione `myFunction`, una variabile `myFunctionProvider` verra generata.

Le funzioni annotate decono specificare "ref" come primo parametro. La funzione è libera di ritornare un `Future/Stream`.

Questa fnìunzione sarà chiamata quando il provider viene letto per la prima volta. L letture successive non chiameranno nuovamente la funzione, ma resituiranno invece il valore memorizzato nella cache.

Ref:

Un oggetto usato per interagire con gli altri provider. Tutti i provider hanno uno; o come parametro della funzone del provider, o come proprietà di un Notifier. Il tipo di questo oggetto è determinato dal nome della funzione/classe.

Usando la sintassi descritta precedentemente possiamo definire il provider:

```dart:

import 'dart:convert';

import 'package:http/http.dart' as http;
import 'package:riverpod_annotation/riverpod_annotation.dart';

import 'activity.dart';

// Necessario affinché la generazione del codice funzioni
part 'provider.g.dart';

/// This will create a provider named `activityProvider`
/// which will cache the result of this function.
@riverpod
Future<Activity> activity(ActivityRef ref) async {
  // Usando il package http, otteniamo un'attività casuale dalle Bored API
  final response = await http.get(Uri.https('boredapi.com', '/api/activity'));
  // Usando dart:convert, decodifichiamo il payload JSON in una Map.
  final json = jsonDecode(response.body) as Map<String, dynamic>;
  // Infine, convertiamo la mappa in un'istanza Activity
  return Activity.fromJson(json);
}
```

In qeusto snippet, abbiamo deiìfinito un provider chiamato activityProvider che potrà essere usata dalla nostra UI per ottenere un'attvità casuale.

- La richiesta di rete non verrà eseguita fino a quando l'UI non leggerà il provider almeno una volta.
- Le letture successive non eseguiranno nuovamente la richiesta di rete, ma restituiranno invece l'attività precedentemente recuperata.
- Se l'UI smette di utilizzare questo provider la cache sarà distrutta
- Non abbiamo gestito gli errori, questo perchè nativamente Riverpod gestisce gli errori e l'UI ha già tutte le informazioni per gestirli

> I provider sono "pigri" (lazy). Definire un provider non eseguirà la richiesta di rete. Invece, la richiesta verrà eseguita alla prima lettura del provider.  


### Visualizzare la risposta della richiesta di rete nell'UI

Ora che abbiamo definito un provider possiamo iniziare ad utiulizzarlo dentro la nostra interfaccia utente per mostrare l'attività

Per interagire con un provider ci serve un oggetto chiamato "ref". Dobbiamo utilizzare un widget personalizzato chiamato `Consumer`. Un Consumer widget è simile a Build ma con il benficio aggiunto di offrirci un oggetto "ref".Ciò permette alla nostra UI di leggere i provider.

Esistono sia i `ConsumerWidget` sia i `ConsumerStatefulWidget`

___

## ESEGUIRE SIDE EFFECTS

Vediamo ora come effettuaer richieste POST

Quando lo fanno, è comune che una richiesta di aggiornamento debba aggiornare anche la cache locale in modo che l'interfaccia utente rifletta il nuovo stato

Di natura, i provider non espongono un modo per modificare il loro stato. Questo è fatto apposta, per garantire che lo stato venga modificato in modo controllato e promuovere la separazione delle responsabilità.
Invece, i provider devono esplicitamente esporre un modo per modificare il loro stato.

Per fare ciò, useremo un nuovo concetto: i __Notifiers__.
Per illustrare questo nuovo concetto, utilizziamo un esempio più avanzato: una lista di cose da fare (to-do list).

Pr aggiungere elementi alla lista dovremmo modifiare i nostri provider in modo che espongano un'API pubblica per modificare il loro stato. Questo si fa convertendo il nostro provider in quello che chiamiamo un "notifier".

I notifier son il "widget stateful" dei provider. Richiedono una piccola modifica alla sintassi per la definizione di un provider. 
La nuova sintssi è la seguente:

```dart:
@riverpod
class MyNotifier extends _$MyNotifier {
  @override
  Result build() {
    <your logic here>
  }
  <your methods here>
}
```

L'annotazione:

Tutti i provider devono essere annotati con `@riverpod`


Il notifier:

Quando un'annotazione `@riverpod` è posta su una classe, quella classe viene chiamata "Notifier". La classe deve estendere `_$NotifierName`, dove `NotifierName` è il nome della classe.

i Notifiers sono responsabili di esporre modalità per modificare lo stato del provider. 
I metodisono pubblici di questa classe sono accessibili ai consumer usando:
`ref.read(yourProvuder.notifier).tuoMetodo()`

> I Notifiers non dovrebbero avere proprietà pubbliche oltre alla proprietà built-in state, poiché l'interfaccia utente non avrebbe modo di sapere che lo stato è cambiato.


Il metodo build:

Tutti i notifiers devono sovrascrivere il metodo `build`.
Questo metodo è equivalente al punto in cui normalmente inseriresti la tua logica in un provider non-notifier

Questo metodo non dovrebbe essere chiamato direttamente.

