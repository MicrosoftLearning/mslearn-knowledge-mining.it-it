---
lab:
  title: Creare una competenza personalizzata per Azure Al Search
  module: Module 12 - Creating a Knowledge Mining Solution
---

# Creare una competenza personalizzata per Azure Al Search

Azure AI Search usa una pipeline di arricchimento delle competenze di intelligenza artificiale per estrarre i campi generati dall'intelligenza artificiale dai documenti e includerli in un indice di ricerca. È disponibile un set completo di competenze incorporate utilizzabile. In presenza di un requisito specifico che non viene soddisfatto da queste competenze, è possibile creare una competenza personalizzata.

Questo esercizio mostra come creare una competenza personalizzata che rappresenta in forma tabulare la frequenza delle singole parole in un documento, per generare un elenco delle prime cinque parole più usate che verrà aggiunto a una soluzione di ricerca per Margie's Travel, un'agenzia di viaggi fittizia.

## Prepararsi allo sviluppo di un'app in Visual Studio Code

Si svilupperà l'app di ricerca usando Visual Studio Code. I file di codice per l'app sono stati forniti in un repository GitHub.

> **Suggerimento**: se è già stato clonato il repository **mslearn-knowledge-mining**, aprirlo in Visual Studio Code. In caso contrario, eseguire i passaggi seguenti per clonarlo nell'ambiente di sviluppo.

1. Avviare Visual Studio Code.
1. Aprire il riquadro comandi (MAIUSC+CTRL+P) ed eseguire un comando **Git: Clone** per clonare il repository `https://github.com/MicrosoftLearning/mslearn-knowledge-mining` in una cartella locale. Non è importante usare una cartella specifica.
1. Dopo la clonazione del repository, aprire la cartella in Visual Studio Code.
1. Attendere il completamento dell'installazione di file aggiuntivi per supportare i progetti in codice C# nel repository.

    > **Nota**: se viene richiesto di aggiungere gli asset necessari per la compilazione e il debug, selezionare **Non adesso**.

## Creazione di risorse Azure

> **Nota**: Se in precedenza è stato completato l'esercizio **[Creare una soluzione di Azure AI Search](01-azure-search.md)** e le risorse di Azure sono ancora disponibili nella sottoscrizione, è possibile ignorare questa sezione e iniziare dalla sezione **Creare una soluzione di ricerca**. In caso contrario, seguire la procedura seguente per effettuare il provisioning delle risorse di Azure necessarie.

1. In un Web browser aprire il portale di Azure all'indirizzo `https://portal.azure.com` ed eseguire l'accesso usando l'account Microsoft associato alla sottoscrizione di Azure.
2. Nella barra di ricerca superiore, cercare *Servizi di Azure AI*, selezionare **Account multiservizio di Servizi di Azure AI** e creare una risorsa account multiservizio di Servizi di Azure AI con le impostazioni seguenti:
    - **Sottoscrizione**: *la sottoscrizione di Azure usata*
    - **Gruppo di risorse**: *scegliere o creare un gruppo di risorse. Se si usa una sottoscrizione con restrizioni, si potrebbe non essere autorizzati a creare un nuovo gruppo di risorse. Usare quello fornito*
    - **Area**: *scegliere tra le aree disponibili geograficamente vicine*
    - **Nome**: *immettere un nome univoco*
    - **Piano tariffario**: Standard S0.
1. Dopo averla distribuita, passare alla risorsa e nella pagina **Panoramica** prendere nota dei valori di **ID sottoscrizione** e **Località**. Questi valori saranno necessari, insieme al nome del gruppo di risorse, nei passaggi successivi. 
1. In Visual Studio Code espandere la cartella **Labfiles/02-search-skill** e selezionare **setup.cmd**. Questo script batch verrà usato per eseguire i comandi dell'interfaccia della riga di comando di Azure che consentono di creare le risorse di Azure necessarie.
1. Fare clic con il pulsante destro del mouse sulla cartella **02-search-skill** e scegliere **Apri nel terminale integrato**.
1. Nel riquadro del terminale immettere il comando seguente per stabilire una connessione autenticata alla sottoscrizione di Azure.

    ```powershell
    az login --output none
    ```

8. Quando richiesto, selezionare o accedere alla propria sottoscrizione Azure. Tornare quindi a Visual Studio Code e attendere il completamento del processo di accesso.
9. Eseguire il comando seguente per elencare le località di Azure.

    ```powershell
    az account list-locations -o table
    ```

10. Nell'output individuare il valore **Name** corrispondente alla località del gruppo di risorse (ad esempio per *Stati Uniti orientali* il nome corrispondente è *eastus*).
11. Nello script **setup.cmd** modificare le dichiarazioni di variabili **subscription_id**, **resource_group** e **location** con i valori appropriati per ID sottoscrizione, nome del gruppo di risorse e nome della località in uso. Salvare quindi le modifiche.
12. Nel terminale per la cartella **02-search-skill** immettere il comando seguente per eseguire lo script:

    ```powershell
    ./setup
    ```

    > **Nota**: Se si verifica un errore dello script, assicurarsi di avere eseguito il salvataggio con i nomi delle variabili corretti e riprovare.

13. Al termine dello script, esaminare l'output visualizzato e prendere nota delle informazioni seguenti sulle risorse di Azure (questi valori saranno necessari in un secondo momento):
    - Nome account di archiviazione
    - Stringa di connessione di archiviazione
    - Endpoint del servizio di ricerca
    - Chiave amministratore del servizio di ricerca
    - Chiave di query del servizio di ricerca

14. Nel portale di Azure aggiornare il gruppo di risorse e verificare che contenga l'account di Archiviazione di Azure, la risorsa di Servizi di Azure AI e la risorsa di Azure AI Search.

## Creare una soluzione di ricerca

Ora che le risorse di Azure necessarie sono disponibili, è possibile creare una soluzione di ricerca costituita dai componenti seguenti:

- Un'**origine dati** che fa riferimento ai documenti nel contenitore di archiviazione di Azure.
- Un **set di competenze** che definisce una pipeline di arricchimento delle competenze per estrarre dai documenti i campi generati dall'intelligenza artificiale.
- Un **indice** che definisce un set di record di documenti in cui è possibile eseguire ricerche.
- Un **indicizzatore** che estrae i documenti dall'origine dati, applica il set di competenze e popola l'indice.

In questo esercizio si userà l'interfaccia REST di Azure AI Search per creare questi componenti inviando richieste JSON.

1. In Visual Studio Code, nella cartella **02-search-skill** espandere la cartella **create-search** e selezionare **data_source.json**. Questo file contiene una definizione JSON per un'origine dati denominata **margies-custom-data**.
2. Sostituire il segnaposto **YOUR_CONNECTION_STRING** con la stringa di connessione per l'account di archiviazione di Azure, che sarà simile alla seguente:

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *È possibile trovare la stringa di connessione nella pagina **Chiavi di accesso** per l'account di archiviazione nel portale di Azure.*

3. Salvare e chiudere il file JSON aggiornato.
4. Nella cartella **create-search** aprire **skillset.json**. Questo file contiene una definizione JSON per un set di competenze denominato **margies-custom-data**.
5. Nella parte superiore della definizione del set di competenze, nell'elemento **cognitiveServices** sostituire il segnaposto **YOUR_AI_SERVICES_KEY** con una delle chiavi per le risorse di Servizi di Azure AI.

    *È possibile trovare le chiavi nella pagina **Chiavi ed endpoint** della risorsa di Servizi di Azure AI nel portale di Azure.*

6. Salvare e chiudere il file JSON aggiornato.
7. Nella cartella **create-search** aprire **index.json**. Questo file contiene una definizione JSON per un indice denominato **margies-custom-index**.
8. Esaminare il codice JSON per l'indice e quindi chiudere il file senza apportare modifiche.
9. Nella cartella **create-search** aprire **indexer.json**. Questo file contiene una definizione JSON per un indicizzatore denominato **margies-custom-indexer**.
10. Esaminare il codice JSON per l'indicizzatore e quindi chiudere il file senza apportare modifiche.
11. Nella cartella **create-search** aprire **create-search.cmd**. Questo script batch usa l'utilità cURL per inviare le definizioni JSON all'interfaccia REST per la risorsa di Azure AI Search.
12. Sostituire le variabili segnaposto **YOUR_SEARCH_URL** e **YOUR_ADMIN_KEY** con l'**URL** e una delle **chiavi amministratore** della risorsa di Azure AI Search.

    *È possibile trovare questi valori nelle pagine **Panoramica** e **Chiavi** per la risorsa di Azure AI Search nel portale di Azure.*

13. Salvare il file batch aggiornato.
14. Fare clic con il pulsante destro del mouse sulla cartella **create-search** e scegliere **Apri nel terminale integrato**.
15. Nel riquadro del terminale per la cartella **create-search** immettere il comando seguente per eseguire lo script batch.

    ```powershell
    ./create-search
    ```

16. Al termine dello script, nella pagina della risorsa di Azure AI Search nel portale di Azure selezionare la pagina **Indicizzatori** e attendere il completamento del processo di indicizzazione.

    *È possibile selezionare **Aggiorna** per tenere traccia dello stato di avanzamento dell'operazione di indicizzazione. Il completamento può richiedere circa un minuto.*

## Eseguire ricerche nell'indice

Ora è possibile eseguire ricerche nell'indice disponibile.

1. Nella parte superiore del pannello della risorsa di Azure AI Search selezionare **Esplora ricerche**.
2. Nella casella **Stringa di query** di Esplora ricerche immettere la stringa di query seguente e quindi selezionare **Cerca**.

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment eq 'positive'
    ```

    La query recupera gli elementi **url**, **sentiment** e **keyphrases** per tutti i documenti in cui è presente la parola *London*, il cui autore è *Reviewer* e che hanno un'etichetta di **sentiment** equivalente a "positive" (in altre parole, recensioni positive che parlano di Londra)

## Creare una funzione di Azure per una competenza personalizzata

La soluzione di ricerca include una serie di competenze di IA incorporate che arricchiscono l'indice con le informazioni dei documenti, ad esempio i punteggi del sentiment e gli elenchi di frasi chiave visualizzate nell'attività precedente.

È possibile migliorare ulteriormente l'indice creando competenze personalizzate. Ad esempio, potrebbe essere utile identificare le parole usate più di frequente in ogni documento, ma non c’è una competenza predefinita che offre questa funzionalità.

Per implementare la funzionalità di conteggio delle parole come competenza personalizzata, si creerà una funzione di Azure nel linguaggio preferito.

> **Nota**: in questo esercizio si creerà un semplice funzione di Node.JS usando le funzionalità di modifica del codice nel portale di Azure. In una soluzione per la fase di produzione si usa in genere un ambiente di sviluppo come Visual Studio Code per creare un'app per le funzioni nel linguaggio preferito (ad esempio C#, Python, Node.JS o Java) e pubblicarla in Azure durante un processo DevOps.

1. Nella pagina **Home** del portale di Azure creare una nuova risorsa **App per le funzioni** con le impostazioni seguenti:
    - **Piano di hosting**: a consumo
    - **Sottoscrizione**: *Sottoscrizione in uso*
    - **Gruppo di risorse**: *lo stesso gruppo di risorse della risorsa di Azure AI Search*
    - **Nome dell'app per le funzioni**: *un nome univoco*
    - **Stack di runtime**: Node.js
    - **Versione**: 18 LTS
    - **Area**: *la stessa area della risorsa di Azure AI Search*
    - **Sistema operativo**: Windows

2. Attendere il completamento della distribuzione e quindi passare alla risorsa App per le funzioni distribuita.
3. Nella parte inferiore della pagina **Panoramica**, selezionare **Crea funzione** per creare una nuova funzione con le impostazioni seguenti:
    - **Selezionare un modello**
        - **Modello**: Trigger HTTP    
    - **Dettagli modello**:
        - **Nome funzione**: wordcount
        - **Livello di autorizzazione**: Funzione
4. Attendere il completamento della creazione della funzione *wordcount*. Nella pagina della funzione selezionare quindi la scheda **Codice e test**.
5. Sostituire il codice predefinito della funzione con il codice seguente:

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    if (req.body && req.body.values) {

        vals = req.body.values;

        // Array of stop words to be ignored
        var stopwords = ['', 'i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 
        "youre", "youve", "youll", "youd", 'your', 'yours', 'yourself', 
        'yourselves', 'he', 'him', 'his', 'himself', 'she', "shes", 'her', 
        'hers', 'herself', 'it', "its", 'itself', 'they', 'them', 
        'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 
        'this', 'that', "thatll", 'these', 'those', 'am', 'is', 'are', 'was',
        'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 
        'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 
        'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 
        'about', 'against', 'between', 'into', 'through', 'during', 'before', 
        'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 
        'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 
        'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 
        'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 
        'only', 'own', 'same', 'so', 'than', 'too', 'very', 'can', 'will',
        'just', "dont", 'should', "shouldve", 'now', "arent", "couldnt", 
        "didnt", "doesnt", "hadnt", "hasnt", "havent", "isnt", "mightnt", "mustnt",
        "neednt", "shant", "shouldnt", "wasnt", "werent", "wont", "wouldnt"];

        res = {"values":[]};

        for (rec in vals)
        {
            // Get the record ID and text for this input
            resVal = {recordId:vals[rec].recordId, data:{}};
            txt = vals[rec].data.text;

            // remove punctuation and numerals
            txt = txt.replace(/[^ A-Za-z_]/g,"").toLowerCase();

            // Get an array of words
            words = txt.split(" ")

            // count instances of non-stopwords
            wordCounts = {}
            for(var i = 0; i < words.length; ++i) {
                word = words[i];
                if (stopwords.includes(word) == false )
                {
                    if (wordCounts[word])
                    {
                        wordCounts[word] ++;
                    }
                    else
                    {
                        wordCounts[word] = 1;
                    }
                }
            }

            // Convert wordcounts to an array
            var topWords = [];
            for (var word in wordCounts) {
                topWords.push([word, wordCounts[word]]);
            }

            // Sort in descending order of count
            topWords.sort(function(a, b) {
                return b[1] - a[1];
            });

            // Get the first ten words from the first array dimension
            resVal.data.text = topWords.slice(0,9)
              .map(function(value,index) { return value[0]; });

            res.values[rec] = resVal;
        };

        context.res = {
            body: JSON.stringify(res),
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
    else {
        context.res = {
            status: 400,
            body: {"errors":[{"message": "Invalid input"}]},
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
};
```

6. Salvare la funzione e quindi aprire il riquadro **Test/Esegui**.
7. Nel riquadro **Test/Esegui** sostituire il **Corpo** esistente con il codice JSON riportato di seguito, che riflette lo schema previsto da una competenza di Azure AI Search in cui i record contenenti dati per uno o più documenti vengono inviati per l'elaborazione:

    ```json
    {
        "values": [
            {
                "recordId": "a1",
                "data":
                {
                "text":  "Tiger, tiger burning bright in the darkness of the night.",
                "language": "en"
                }
            },
            {
                "recordId": "a2",
                "data":
                {
                "text":  "The rain in spain stays mainly in the plains! That's where you'll find the rain!",
                "language": "en"
                }
            }
        ]
    }
    ```

8. Fare clic su **Esegui** e visualizzare il contenuto della risposta HTTP restituito dalla funzione. Questo riflette lo schema previsto da Azure AI Search quando si utilizza una competenza, in cui viene restituita una risposta per ogni documento. In tal caso, la risposta è costituita da un massimo di 10 termini in ogni documento, in ordine decrescente in base alla frequenza:

    ```json
    {
        "values": [
        {
            "recordId": "a1",
            "data": {
                "text": [
                "tiger",
                "burning",
                "bright",
                "darkness",
                "night"
                ]
            }
        },
        {
            "recordId": "a2",
            "data": {
                "text": [
                    "rain",
                    "spain",
                    "stays",
                    "mainly",
                    "plains",
                    "thats",
                    "youll",
                    "find"
                ]
            }
        }
        ]
    }
    ```

9. Chiudere il riquadro **Test/Esegui** e fare clic su **Recupera URL della funzione** nel pannello della funzione **wordcount**. Copiare quindi l'URL della chiave predefinita negli Appunti. Sarà necessario nella procedura successiva.

## Aggiungere la competenza personalizzata alla soluzione di ricerca

A questo punto è necessario includere la funzione come competenza personalizzata nel set di competenze della soluzione di ricerca ed eseguire il mapping dei risultati prodotti a un campo nell'indice. 

1. Nella cartella **02-search-skill/update-search** di Visual Studio Code aprire il file **update-skillset.json**. Contiene la definizione JSON di un set di competenze.
2. Esaminare la definizione del set di competenze. Include le stesse competenze di prima e una nuova competenza **WebApiSkill** denominata **get-top-words**.
3. Modificare la definizione della competenza **get-top-words** per impostare il valore **uri** sull'URL della funzione di Azure copiato negli Appunti durante la procedura precedente, sostituendo **YOUR-FUNCTION-APP-URL**.
4. Nella parte superiore della definizione del set di competenze, nell'elemento **cognitiveServices** sostituire il segnaposto **YOUR_AI_SERVICES_KEY** con una delle chiavi per le risorse di Servizi di Azure AI.

    *È possibile trovare le chiavi nella pagina **Chiavi ed endpoint** della risorsa di Servizi di Azure AI nel portale di Azure.*

5. Salvare e chiudere il file JSON aggiornato.
6. Nella cartella **update-search** aprire **update-index.json**. Questo file contiene la definizione JSON dell'indice **margies-custom-index**, con un campo aggiuntivo denominato **top_words** nella parte inferiore della definizione dell'indice.
7. Esaminare il codice JSON per l'indice e quindi chiudere il file senza apportare modifiche.
8. Nella cartella **update-search** aprire **update-indexer.json**. Questo file contiene una definizione JSON per **margies-custom-indexer**, con un mapping aggiuntivo per il campo personalizzato **top_words**.
9. Esaminare il codice JSON per l'indicizzatore e quindi chiudere il file senza apportare modifiche.
10. Nella cartella **update-search** aprire **update-search.cmd**. Questo script batch usa l'utilità cURL per inviare le definizioni JSON aggiornate all'interfaccia REST per la risorsa di Azure AI Search.
11. Sostituire le variabili segnaposto **YOUR_SEARCH_URL** e **YOUR_ADMIN_KEY** con l'**URL** e una delle **chiavi amministratore** della risorsa di Azure AI Search.

    *È possibile trovare questi valori nelle pagine **Panoramica** e **Chiavi** per la risorsa di Azure AI Search nel portale di Azure.*

12. Salvare il file batch aggiornato.
13. Fare clic con il pulsante destro del mouse sulla cartella **update-search** e scegliere **Apri nel terminale integrato**.
14. Nel riquadro del terminale per la cartella **update-search** immettere il comando seguente per eseguire lo script batch.

    ```powershell
    ./update-search
    ```

15. Al termine dello script, nella pagina della risorsa di Azure AI Search nel portale di Azure selezionare la pagina **Indicizzatori** e attendere il completamento del processo di indicizzazione.

    *È possibile selezionare **Aggiorna** per tenere traccia dello stato di avanzamento dell'operazione di indicizzazione. Il completamento può richiedere circa un minuto.*

## Eseguire ricerche nell'indice

Ora è possibile eseguire ricerche nell'indice disponibile.

1. Nella parte superiore del pannello della risorsa di Azure AI Search selezionare **Esplora ricerche**.
2. In Esplora ricerche passare alla **visualizzazione JSON**e quindi inviare la query di ricerca seguente:

    ```json
    {
      "search": "Las Vegas",
      "select": "url,top_words"
    }
    ```

    Questa query recupera i campi **url** e **top_words** per tutti i documenti che menzionano *Las Vegas*.

## Eliminazione

Dopo aver completato l'esercizio, eliminare tutte le risorse non più necessarie. Eliminare le risorse di Azure:

1. Nel portale di Azure, selezionare **Gruppi di risorse**.
1. Selezionare il gruppo di risorse non necessario, quindi selezionare **Elimina gruppo di risorse**.

## Ulteriori informazioni

Per altre informazioni sulla creazione di competenze personalizzate per Azure AI Search, vedere la [documentazione di Azure AI Search](https://docs.microsoft.com/azure/search/cognitive-search-custom-skill-interface).
