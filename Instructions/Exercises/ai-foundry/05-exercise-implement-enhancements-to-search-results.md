---
lab:
  title: Implementare miglioramenti per i risultati della ricerca
---

# Implementare miglioramenti per i risultati della ricerca

È disponibile un servizio di ricerca usato da un'app di prenotazione di vacanze. Si è visto che la pertinenza dei risultati della ricerca influisce sul numero di prenotazioni ricevute. Di recente sono stati inoltre aggiunti hotel in Portogallo, per cui si desidera offrire il portoghese come lingua supportata.

In questo esercizio si aggiungerà un profilo di punteggio per migliorare la pertinenza dei risultati della ricerca. Si useranno quindi i servizi di Azure AI per aggiungere descrizioni in portoghese per tutti gli hotel.

> **Nota** Per completare questo esercizio, sarà necessaria una sottoscrizione di Microsoft Azure. Se non è ancora disponibile alcuna sottoscrizione, è possibile registrarsi per una valutazione gratuita all'indirizzo [https://azure.com/free](https://azure.com/free?azure-portal=true).

## Creazione di risorse Azure

Si creerà un servizio Azure AI Search e si importeranno dati di hotel di esempio.

1. Accedere al [portale di Azure](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true).
1. Selezionare **+ Crea una risorsa**.
1. Cercare **Ricerca**, quindi selezionare **Azure AI Search**.
1. Seleziona **Crea**.
1. Selezionare **Crea nuovo** in Gruppo di risorse, denominarlo **learn-advanced-search**.
1. In **Nome servizio** immettere **advanced-search-service-12345**. Il nome deve essere univoco a livello globale, quindi aggiungere numeri casuali alla fine.
1. Selezionare un'area supportata nelle vicinanze.
1. Usare i valori predefiniti per **Piano tariffario**.
1. Selezionare **Rivedi e crea**.
1. Seleziona **Crea**.
1. Attendere la distribuzione delle risorse e quindi selezionare **Vai alla risorsa**.

### Importare dati di esempio nel servizio di ricerca

Importare i dati di esempio.

1. Nel riquadro **Panoramica** selezionare **Importa dati**.

    ![Screenshot del menu Importa dati.](../media/05-media/import-data-new.png)
1. Nel riquadro **Importa dati**, nell'elenco a discesa **Origine dati** selezionare **Esempi**.
1. Selezionare **hotels-sample**.

1. Nella scheda **Aggiungi competenze cognitive (facoltativo)** espandere **Collega Servizi AI** e quindi selezionare **Crea nuova risorsa Servizi di AI**.

    ![Screenshot della selezione e dell'aggiunta di Servizi di Azure AI.](../media/05-media/add-cognitive-services-new.png)

### Creare un servizio Azure AI per supportare le traduzioni

1. Nella nuova scheda accedere al portale di Azure.
1. In **Gruppo di risorse** selezionare **learn-advanced-search**.
1. In **Area** selezionare la stessa area scelta per il servizio di ricerca.
1. In **Nome** immettere **learn-cognitive-translator-12345** o un nome a piacere. Il nome deve essere univoco a livello globale, quindi aggiungere numeri casuali alla fine.
1. In **Piano tariffario** selezionare **Standard S0**.
1. Selezionare **Selezionando questa casella, confermo di aver letto e compreso tutte le condizioni seguenti**.
1. Selezionare **Rivedi e crea**.
1. Seleziona **Crea**.
1. Dopo aver creato le risorse, chiudere la scheda.

### Aggiungere un arricchimento della traduzione

1. Nella scheda **Aggiungi competenze cognitive (facoltativo)** selezionare Aggiorna.
1. Selezionare il nuovo servizio **learn-cognitive-translator-12345**.
1. Espandere la sezione **Aggiungi arricchimenti**.
    ![Screenshot dell'aggiunta della traduzione in portoghese.](../media/05-media/add-translation-enrichment-new.png)
1. Selezionare **Traduci testo**, impostare **Lingua di destinazione** su **Portoghese**, quindi impostare **Nome campo** su **Description_pt**.
1. Selezionare **Avanti: Personalizza indice di destinazione**.

### Cambiare il campo in cui archiviare il testo tradotto

1. Nella scheda **Personalizza indice di destinazione** scorrere fino alla fine dell'elenco dei campi e impostare **Analizzatore** su **Portoghese (Portogallo) - Microsoft** in corrispondenza del campo **Description_pt**.
1. Seleziona **Successivo: Crea un indicizzatore**.
1. Selezionare **Invia**.

    L'indice viene creato, l'indicizzatore verrà eseguito e verranno importati 50 documenti contenenti dati di hotel di esempio.
1. Nel riquadro **Panoramica** selezionare **Indici** e quindi **hotels-sample-index**.
1. Selezionare **Cerca** per visualizzare il codice JSON per tutti i documenti nell'indice.
1. Cercare **Description_pt** (è possibile usare **CTRL+F**) nei risultati e notare che non è una traduzione in portoghese della descrizione inglese, ma è simile a quanto segue:

    ```json
    "Description_pt": "45",
    ```

Il portale di Azure presuppone che sia necessario tradurre il primo campo del documento. Quindi attualmente usa la competenza di traduzione per tradurre `HotelId`.

### Aggiornare il set di competenze per tradurre il campo corretto nel documento

1. Nella parte superiore della pagina selezionare il servizio di ricerca, il collegamento **advanced-search-service-12345 |Indexes**.
1. Selezionare **Set di competenze** in Gestione della ricerca nel riquadro sinistro, quindi selezionare **hotels-sample-skillset**.
1. Modificare il documento JSON, cambiare la riga 11 in:

    ```json
    "context": "/document/Description",
    ```

1. Cambiare il valore predefinito di language in English nella riga 12:

    ```json
    "defaultFromLanguageCode": "en",
    ```

1. Cambiare il campo di origine nella riga 18 in:

    ```json
    "source": "/document/Description"
    ```

1. Seleziona **Salva**.
1. Nella parte superiore della pagina selezionare il servizio di ricerca, il collegamento **advanced-search-service-12345 | Skillsets**.
1. Nel riquadro **Panoramica** selezionare **Indicizzatori** e quindi **hotels-sample-indexer**.
1. Selezionare **Definizione indicizzatore (JSON)**.
1. Cambiare il nome del campo di origine nella riga 21 in:

    ```json
    "sourceFieldName": "/document/Description/Description_pt",
    ```

1. Seleziona **Salva**.
1. Selezionare **Reimposta**, quindi **Sì**.
1. Selezionare **Esegui**, quindi **Sì**.

### Testare l'indice aggiornato

1. Nella parte superiore della pagina selezionare il servizio di ricerca, il collegamento **advanced-search-service-12345 | Indexers**.
1. Nel riquadro **Panoramica** selezionare **Indici** e quindi **hotels-sample-index**.
1. Selezionare **Cerca** per visualizzare il codice JSON per tutti i documenti nell'indice.
1. Cercare **Description_pt** nei risultati e notare che ora è presente una descrizione in portoghese.

    ```json
    "Description_pt": "O maior resort durante todo o ano da área oferecendo mais de tudo para suas férias – pelo melhor valor!  O que você pode desfrutar enquanto estiver no resort, além das praias de areia de 1,5 km do lago? Confira nossas atividades com certeza para excitar tanto os jovens quanto os jovens hóspedes do coração. Temos tudo, incluindo ser chamado de \"Propriedade do Ano\" e um \"Top Ten Resort\" pelas principais publicações.",
    ```

1. Cercare ora hotel vista lago. Si inizierà usando una ricerca semplice che restituisce solo `HotelName`, `Description`, `Category` e `Tags`. In **Stringa di query** immettere questa ricerca:

    `lake + view&$select=HotelName,Description,Category,Tags&$count=true`

    Esaminare i risultati e provare a individuare i campi corrispondenti ai termini di ricerca `lake`e `view`. Notare questo hotel e la relativa posizione:

    ```json
    {
      "@search.score": 0.9433406,
      "HotelName": "Lady Of The Lake B & B",
      "Description": "Nature is Home on the beach.  Save up to 30 percent. Valid Now through the end of the year. Restrictions and blackout may apply.",
      "Category": "Luxury",
      "Tags": [
        "laundry service",
        "concierge",
        "view"
      ]
    },
    ```

Per questo hotel sono state trovate corrispondenze per il termine lake nel campo `HotelName` e per view nel campo `Tags`. Si vogliono aumentare le corrispondenze di termini nel campo `Description` rispetto al nome dell'hotel. Idealmente, questo hotel dovrebbe essere l'ultimo nei risultati.

## Aggiungere un profilo di punteggio per migliorare i risultati della ricerca

1. Selezionare la scheda **Profili di punteggio**.
1. Selezionare **+ Aggiungi profilo di punteggio**.
1. In **Nome profilo** immettere **boost-description-categories**.
1. Aggiungere i campi e i pesi seguenti in **Pesi**:

    ![Screenshot dell'aggiunta di pesi a un profilo di punteggio.](../media/05-media/add-weights-new.png)
1. In **Nome campo** selezionare **Descrizione**.
1. In **Peso** immettere **5**.
1. In **Nome campo** selezionare **Categoria**.
1. In **Peso** immettere **3**.
1. In **Nome campo** selezionare **Tag**.
1. In **Peso** immettere **2**.
1. Seleziona **Salva**.
1. Seleziona **Salva** in alto.

### Testare l'indice aggiornato

1. Tornare alla scheda **Esplora ricerche** della pagina **hotels-sample-index**.
1. In **Stringa di query** immettere la stessa ricerca di prima:

    `lake + view&$select=HotelName,Description,Category,Tags&$count=true`

    Controllare i risultati della ricerca.

    ```json
    {
      "@search.score": 3.5707965,
      "HotelName": "Lady Of The Lake B & B",
      "Description": "Nature is Home on the beach.  Save up to 30 percent. Valid Now through the end of the year. Restrictions and blackout may apply.",
      "Category": "Luxury",
      "Tags": [
        "laundry service",
        "concierge",
        "view"
      ]
    }
    ```

    Il punteggio di ricerca è aumentato da **0,9433406** a **3,5707965**. Tuttavia, tutti gli altri hotel hanno punteggi calcolati superiori. Questo hotel è ora l'ultimo nei risultati.

## Eliminazione

Dopo aver completato l'esercizio, eliminare tutte le risorse non più necessarie.

1. Nel portale di Azure, selezionare **Gruppi di risorse**.
1. Selezionare il gruppo di risorse che non è più necessario, quindi selezionare **Elimina gruppo di risorse**.
