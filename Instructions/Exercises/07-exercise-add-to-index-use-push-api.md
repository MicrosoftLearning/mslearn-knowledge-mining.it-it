---
lab:
  title: Aggiungere elementi a un indice usando l'API push
---

# Aggiungere elementi a un indice usando l'API push

Si vuole esplorare come creare un indice di Azure AI Search e caricare documenti in tale indice usando il codice C#.

In questo esercizio si clonerà una soluzione C# esistente e la si eseguirà per determinare le dimensioni ottimali del batch per il caricamento di documenti. Si useranno quindi tali dimensioni del batch e si caricheranno i documenti in modo efficace usando un approccio in thread.

> **Nota** Per completare questo esercizio, sarà necessaria una sottoscrizione di Microsoft Azure. Se non è ancora disponibile alcuna sottoscrizione, è possibile registrarsi per una valutazione gratuita all'indirizzo [https://azure.com/free](https://azure.com/free?azure-portal=true).

## Configurare le risorse di Azure

Per risparmiare tempo, selezionare questo modello di Azure Resource Manager per creare le risorse che saranno necessarie più avanti nell'esercizio:

1. [Distribuire risorse in Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoftLearning%2Fmslearn-knowledge-mining%2Fmain%2FLabfiles%2F07-exercise-add-to-index-use-push-api%20lab-files%2Fazuredeploy.json): selezionare questo collegamento per creare risorse di Azure per intelligenza artificiale.
    ![Screenshot delle opzioni visualizzate durante la distribuzione delle risorse in Azure.](../media/07-media/deploy-azure-resources.png)
1. In **Gruppo di risorse** selezionare **Crea nuovo** e assegnare il nome **cog-search-language-exe**.
1. In **Area** selezionare un'[area supportata](/azure/ai-services/language-service/custom-text-classification/service-limits#regional-availability) nelle vicinanze.
1. Il valore di **Prefisso della risorsa** deve essere univoco a livello globale: immettere un prefisso casuale con caratteri minuscoli e numeri, ad esempio **acs118245**.
1. In **Località** selezionare la stessa area scelta in precedenza.
1. Selezionare **Rivedi e crea**.
1. Seleziona **Crea**.
1. Al termine della distribuzione, selezionare **Vai al gruppo di risorse** per visualizzare tutte le risorse create.

    ![Screenshot che mostra tutte le risorse di Azure distribuite.](../media/07-media/azure-resources-created.png)

## Copiare le informazioni dell'API REST del servizio Azure AI Search

1. Nell'elenco di risorse selezionare il servizio di ricerca creato. Nell'esempio precedente, **acs118245-search-service**.
1. Copiare il nome del servizio di ricerca in un file di testo.

    ![Screenshot della sezione Chiavi di un servizio di ricerca.](../media/07-media/search-api-keys-exercise-version.png)
1. A sinistra selezionare **Chiavi** e quindi copiare il valore di **Chiave di amministrazione primaria** nello stesso file di testo.

## Clonare il repository in Cloud Shell

Verrà sviluppato il codice usando Cloud Shell dal portale di Azure. I file di codice per l'app sono stati forniti in un repository GitHub.

> **Suggerimento**: se il repository **mslearn-knowledge-mining** è già stato clonato di recente, è possibile saltare questa attività. In caso contrario, eseguire i passaggi seguenti per clonarlo nell'ambiente di sviluppo.

1. Nel portale di Azure, usare il pulsante **[\>_]** a destra della barra di ricerca nella parte superiore della pagina per creare una nuova Cloud Shell nel portale di Azure, selezionando un ambiente ***PowerShell***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *Bash*, passare a ***PowerShell***.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

    > **Suggerimento**: quando si incollano i comandi in CloudShell, l'ouput può richiedere una grande quantità di buffer dello schermo. È possibile cancellare la schermata immettendo il `cls` comando per rendere più semplice concentrarsi su ogni attività.

1. Nel riquadro PowerShell immettere i comandi seguenti per clonare il repository GitHub per questo esercizio:

    ```
    rm -r mslearn-knowledge-mining -f
    git clone https://github.com/microsoftlearning/mslearn-knowledge-mining mslearn-knowledge-mining
    ```

1. Dopo aver clonato il repository, passare alla cartella contenente i file di codice dell'applicazione:  

    ```
   cd mslearn-knowledge-mining/Labfiles/07-exercise-add-to-index-use-push-api/OptimizeDataIndexing
    ```

## Configurazione dell'applicazione

1. Con il comando `ls`, è possibile visualizzare il contenuto della cartella **OptimizeDataIndexing**. Notare che contiene un file `appsettings.json` per le impostazioni di configurazione.

1. Immettere il comando seguente per modificare il file di configurazione fornito:

    ```
   code appsettings.json
    ```

    Il file viene aperto in un editor di codice.

    ![Screenshot che mostra il contenuto del file appsettings.json.](../media/07-media/update-app-settings.png)

1. Incollare il nome del servizio di ricerca e la chiave amministratore primaria.

    ```json
    {
      "SearchServiceUri": "https://acs118245-search-service.search.windows.net",
      "SearchServiceAdminApiKey": "YOUR_SEARCH_SERVICE_KEY",
      "SearchIndexName": "optimize-indexing"
    }
    ```

    Il file delle impostazioni sarà simile a quello riportato sopra.
   
1. Dopo aver sostituito i segnaposto, usare il comando **CTRL+S** per salvare le modifiche e quindi usare il comando **CTRL+Q** per chiudere l'editor di codice mantenendo aperta la riga di comando di Cloud Shell.
1. Nel terminale immettere `dotnet run` e premere **Invio**.

    ![Screenshot che mostra l'app in esecuzione in VS Code con un'eccezione.](../media/07-media/debug-application.png)

    L'output mostra che in questo caso la dimensione del batch più performante è 900 documenti con la massima velocità di trasferimento (MB/secondo).
   
    >**Nota**: i valori della velocità di trasferimento potrebbero variare rispetto a quelli mostrati nell'immagine. Tuttavia, la dimensione del batch più performante dovrebbe essere sempre la stessa. 

## Modificare il codice per implementare il threading e una strategia di backoff e retry

È presente codice impostato come commento pronto per modificare l'app in modo da usare thread per caricare i documenti nell'indice di ricerca.

1. Immettere il comando seguente per aprire il file di codice dell'applicazione client:

    ```
   code Program.cs
    ```

1. Impostare come commento le righe 38 e 39 nel seguente modo:

    ```csharp
    //Console.WriteLine("{0}", "Finding optimal batch size...\n");
    //await TestBatchSizesAsync(searchClient, numTries: 3);
    ```

1. Rimuovere il commento per le righe da 41 a 49.

    ```csharp
    long numDocuments = 100000;
    DataGenerator dg = new DataGenerator();
    List<Hotel> hotels = dg.GetHotels(numDocuments, "large");

    Console.WriteLine("{0}", "Uploading using exponential backoff...\n");
    await ExponentialBackoff.IndexDataAsync(searchClient, hotels, 1000, 8);

    Console.WriteLine("{0}", "Validating all data was indexed...\n");
    await ValidateIndexAsync(indexClient, indexName, numDocuments);
    ```

    Il codice che controlla le dimensioni del batch e il numero di thread è `await ExponentialBackoff.IndexDataAsync(searchClient, hotels, 1000, 8)`. Le dimensioni del batch sono pari a 1000 e i thread sono otto.

    ![Screenshot che mostra tutto il codice modificato.](../media/07-media/thread-code-ready.png)

    Il codice sarà simile a quello riportato sopra.

1. Salva le modifiche.
1. Selezionare il terminale, quindi premere un tasto qualsiasi per terminare il processo in esecuzione, se non è già stato fatto.
1. Eseguire `dotnet run` nel terminale.

    L'app avvierà otto thread e quindi un nuovo thread man mano che ogni thread finisce di scrivere un nuovo messaggio nella console:

    ```powershell
    Finished a thread, kicking off another...
    Sending a batch of 1000 docs starting with doc 57000...
    ```

    Dopo il caricamento di 100.000 documenti, l'app scrive un riepilogo (il completamento potrebbe richiedere alcuni minuti):

    ```powershell
    Ended at: 9/1/2023 3:25:36 PM
    
    Upload time total: 00:01:18:0220862
    Upload time per batch: 780.2209 ms
    Upload time per document: 0.7802 ms
    
    Validating all data was indexed...
    
    Waiting for service statistics to update...
    
    Document Count is 100000
    
    Waiting for service statistics to update...
    
    Index Statistics: Document Count is 100000
    Index Statistics: Storage Size is 71453102
    
    ``````

Esplorare il codice nella procedura `TestBatchSizesAsync` per capire come vengono testate le prestazioni delle dimensioni del batch.

Esplorare il codice nella procedura `IndexDataAsync` per capire come viene gestito il threading.

Esplorare il codice in `ExponentialBackoffAsync` per capire come viene implementata una strategia di retry con backoff esponenziale.

È possibile cercare i documenti e verificare che siano stati aggiunti all'indice nel portale di Azure.

![Screenshot che mostra l'indice di ricerca con 100.000 documenti.](../media/07-media/check-search-service-index.png)

## Eliminazione

Dopo aver completato l'esercizio, eliminare tutte le risorse non più necessarie. Iniziare con il codice clonato nel computer. Eliminare quindi le risorse di Azure.

1. Nel portale di Azure, selezionare **Gruppi di risorse**.
1. Selezionare il gruppo di risorse creato per questo esercizio.
1. Selezionare **Elimina gruppo di risorse**. 
1. Confermare l'eliminazione, quindi selezionare **Elimina**.
1. Selezionare le risorse non necessarie e quindi selezionare **Elimina**.
