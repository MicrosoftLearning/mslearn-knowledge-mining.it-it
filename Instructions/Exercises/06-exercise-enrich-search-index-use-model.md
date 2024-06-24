---
lab:
  title: Arricchire un indice di ricerca usando il modello di Azure Machine Learning
---

# Arricchire un indice di ricerca usando il modello di Azure Machine Learning

È possibile usare la potenza di Machine Learning per arricchire un indice di ricerca. A tale scopo, si userà un modello sottoposto a training in studio di AI Azure Machine Learning e lo si chiamerà da un set di competenze personalizzato di Machine Learning.

In questo esercizio si creerà un modello di studio di AI Azure Machine Learning, quindi si eseguirà il training, la distribuzione e il test di un endpoint usando il modello. Si creerà quindi un servizio di ricerca cognitiva di Azure, si creeranno dati di esempio e si arricchirà un indice usando l'endpoint di studio di AI Azure Machine Learning.

> **Nota** Per completare questo esercizio, sarà necessaria una sottoscrizione di Microsoft Azure. Se non è ancora disponibile alcuna sottoscrizione, è possibile registrarsi per una valutazione gratuita all'indirizzo [https://azure.com/free](https://azure.com/free?azure-portal=true).
>

## Creare un'area di lavoro di Azure Machine Learning

Prima di arricchire l'indice di ricerca, creare un'area di lavoro di Azure Machine Learning. L'area di lavoro consentirà di accedere a studio di Azure per intelligenza artificiale Machine Learning, uno strumento grafico che è possibile usare per creare modelli di intelligenza artificiale e distribuirli per l'uso.

1. Accedere al [portale di Azure](https://portal.azure.com).
1. Selezionare **+ Crea una risorsa**.
1. Cercare Machine Learning e quindi selezionare **Azure Machine Learning**.
1. Seleziona **Crea**.
1. Selezionare **Crea nuovo** in **Gruppo di risorse** e denominarlo **aml-for-acs-enrichment**.
1. Nella sezione Dettagli area di lavoro immettere **aml-for-acs-workspace** in **Nome**.
1. Selezionare un'**area** supportata nelle vicinanze.
1. Usare i valori predefiniti per l'**account di archiviazione**, **Insieme di credenziali delle chiavi**, **Application Insights** e **Registro Azure Container**.
1. Selezionare **Rivedi e crea**.
1. Seleziona **Crea**.
1. Attendere la distribuzione dell'area di lavoro di Azure Machine Learning e quindi selezionare **Vai alla risorsa**.
1. Nel riquadro Panoramica selezionare **Avvia Studio**.

## Creare una pipeline di training di regressione

A questo punto si creerà un modello di regressione ed eseguirà il training usando una pipeline di studio di Azure per intelligenza artificiale Machine Learning. Si eseguirà il training del modello sui dati sui prezzi delle automobili. Dopo il training, il modello prevederà il prezzo di un'automobile in base ai relativi attributi.

1. Nella home page selezionare **Progettazione**.

1. Nell'elenco dei componenti predefiniti selezionare **Regressione - Automobile Price Prediction (Basic)**.

    ![Screenshot che mostra la selezione del modello di regressione predefinito.](../media/06-media/select-pre-built-components-new.png)

1. Selezionare **Convalida**.

1. Nel riquadro **Convalida grafico**, selezionare l'errore **Selezionare la destinazione di calcolo nella procedura guidata all’invio**.

    ![Screenshot che mostra come creare un'istanza di calcolo per eseguire il training del modello.](../media/06-media/create-compute-instance-new.png)
1. Nell'elenco a discesa **Seleziona tipo di calcolo**, scegliere **Istanza di calcolo**. Selezionare quindi **Crea istanza di calcolo di Azure ML** sotto.
1. Nel campo **Nome calcolo** immettere un nome univoco, ad esempio **compute-for-training**.
1. Selezionare **Rivedi e crea** e quindi **Crea**.

1. Nel campo **Selezionare l'istanza di calcolo di Azure ML**, selezionare l'istanza dall'elenco a discesa. Potrebbe essere necessario attendere il completamento del provisioning.

1. Selezionare di nuovo **Convalida**. La pipeline dovrebbe avere un aspetto ottimale.

    ![Screenshot che mostra l'aspetto positivo della pipeline e il pulsante Invia evidenziato.](../media/06-media/submit-pipeline.png)
1. Selezionare **Informazioni di base** nel riquadro **Configura processo della pipeline**.
1. Selezionare **Crea nuovo** sotto il nome dell'esperimento.
1. In **Nuovo nome esperiment** immettere **linear-regression-training**.
1. Selezionare **Rivedi e invia** e quindi **Invia**.

### Creare un cluster di inferenza per l'endpoint

Mentre la pipeline esegue il training di un modello di regressione lineare, è possibile creare le risorse necessarie per l'endpoint. Questo endpoint richiede un cluster Kubernetes per elaborare le richieste Web al modello.

1. A sinistra selezionare **Calcolo**.

    ![Screenshot che mostra come creare un nuovo cluster di inferenza.](../media/06-media/create-inference-cluster-new.png)
1. Selezionare **Cluster Kubernetes** e quindi **+ Nuovo**.
1. Nell'elenco a discesa selezionare **AksCompute**.
1. Nel riquadro **Crea AksCompute** selezionare **Crea nuovo**.
1. Per **Località**, selezionare la stessa area usata per creare le altre risorse.
1. Nell'elenco Dimensioni macchina virtuale, selezionare **Standard_A2_v2**.
1. Selezionare **Avanti**.
1. In **Nome calcolo** immettere **aml-acs-endpoint**.
1. Selezionare **Abilita configurazione SSL**.
1. In **Dominio foglia** immettere **aml-for-acs**.
1. Seleziona **Crea**.

### Registrare il modello sottoposto a training

Il processo della pipeline dovrebbe essere terminato. I file `score.py` e `conda_env.yaml` verranno scaricati. Si registrerà quindi il modello sottoposto a training.

1. A sinistra selezionare **Processi**.

    ![Screenshot che mostra il processo della pipeline completato.](../media/06-media/completed-pipeline-new.png)
1. Selezionare l'esperimento, quindi selezionare il processo completato nella tabella, ad esempio **Regressione - Previsioni dei prezzi delle automobili (Basic)**. Se viene richiesto di salvare le modifiche, selezionare **Ignora** per le modifiche.
1. Nella finestra di progettazione selezionare **Panoramica del processo** in alto a destra, quindi selezionare il nodo **Esegui training modello**.

    ![Screenshot che mostra come scaricare score.py.](../media/06-media/download-score-conda.png)
1. Nella scheda **Output e log** espandere la cartella **trained_model_outputs**.
1. Accanto a `score.py`, selezionare il menu altro (**...**), quindi selezionare **Scarica**.
1. Accanto a `conda_env.yaml`, selezionare il menu altro (**...**), quindi selezionare **Scarica**.
1. Selezionare **+ Registra modello** nella parte superiore della scheda.
1. Nel campo **Output processo** selezionare la cartella **trained_model_outputs**. Selezionare quindi **Avanti** nella parte inferiore del riquadro.
1. In **Nome** modello, immettere **carevalmodel**.
1. In **Descrizione**, immettere **Un modello di regressione lineare per stimare il prezzo delle automobili.**.
1. Selezionare **Avanti**.
1. Selezionare **Registra**.

### Modificare lo script di assegnazione dei punteggi per rispondere correttamente a Azure AI Search

Studio di Azure Machine Learning ha scaricato due file nel percorso di download predefinito del Web browser. È necessario modificare il file score.py per modificare la modalità di gestione della richiesta e della risposta JSON. È possibile usare un editor di testo o un editor di codice come Visual Studio Code.

1. Nell'editor, aprire il file score.py.
1. Sostituire tutto il contenuto della funzione run:

    ```python
    def run(data):
    data = json.loads(data)
    input_entry = defaultdict(list)
    for row in data:
        for key, val in row.items():
            input_entry[key].append(decode_nan(val))

    data_frame_directory = create_dfd_from_dict(input_entry, schema_data)
    score_module = ScoreModelModule()
    result, = score_module.run(
        learner=model,
        test_data=DataTable.from_dfd(data_frame_directory),
        append_or_result_only=True)
    return json.dumps({"result": result.data_frame.values.tolist()})
    ```

    Con questo codice Python:

    ```python
    def run(data):
        data = json.loads(data)
        input_entry = defaultdict(list)
        
        for key, val in data.items():
                input_entry[key].append(decode_nan(val))
    
        data_frame_directory = create_dfd_from_dict(input_entry, schema_data)
        score_module = ScoreModelModule()
        result, = score_module.run(
            learner=model,
            test_data=DataTable.from_dfd(data_frame_directory),
            append_or_result_only=True)
        output = result.data_frame.values.tolist()
        
        return {
                "predicted_price": output[0][-1]
        }    
    ```

    Le modifiche precedenti consentono alla modalità di ricevere un singolo oggetto JSON con attributi auto anziché una matrice di automobili.

    L'altra modifica consiste nel restituire solo il prezzo stimato dell'auto anziché l'intera risposta JSON.
1. Salvare le modifiche nell'editor di testo.

## Creare un ambiente personalizzato

Si creerà quindi un ambiente personalizzato in modo da poter essere distribuito in un endpoint in tempo reale.

1. Nel riquadro di spostamento seleziona **Ambienti**.
1. Selezionare la scheda **Ambienti personalizzati**.
1. Seleziona **+ Crea**.
1. In **Nome**, immettere **my-custom-environment**.
1. Nell'elenco degli *ambienti curati* in **Seleziona tipo di ambiente**, selezionare la versione più recente di **automl-gpu**.
1. Selezionare **Avanti**.
1. Nel computer locale aprire il file `conda_env.yaml` scaricato in precedenza e copiarne il contenuto.
1. Tornare al browser e selezionare **conda_dependencies.yaml** nel riquadro Personalizza.
1. Nel riquadro a destra sostituire il relativo contenuto con il codice copiato in precedenza.
1. Selezionare **Avanti** e quindi di nuovo **Avanti**.
1. Selezionare **Crea** per creare l'ambiente personalizzato.

## Distribuire il modello con il codice di assegnazione dei punteggi aggiornato <!--Option for web service deployment is greyed out. Can't go further after trying several different things.-->

Il cluster di inferenza dovrebbe ora essere pronto per l'uso. È stato modificato anche il codice di assegnazione dei punteggi per gestire le richieste dal set di competenze personalizzato di Ricerca cognitiva di Azure. Verrà ora creato e testato un endpoint per il modello.

1. A sinistra selezionare **Modelli**.
1. Selezionare il modello registrato **carevalmodel**.

1. Selezionare **Distribuisci**, e quindi **endpoint in tempo reale**.

    ![Screenshot del riquadro Seleziona endpoint.](../media/06-media/04-select-endpoint.png)
1. In **Nome**, immettere un nome univoco, ad esempio **car-evaluation-endpoint-1440637584**.
1. Per **Tipo di calcolo** selezionare **Gestito**.
1. Per **Tipo di autenticazione** selezionare **Autenticazione**basata su chiave.
1. Selezionare **Avanti** e quindi **Avanti**.
1. Selezionare di nuovo **Avanti**.
1. Nel campo **Selezionare uno script di assegnazione dei punteggi per l'inferenza** passare al file aggiornato `score.py` e selezionarlo.
1. Nell'elenco a discesa **Seleziona tipo di ambiente** selezionare **Ambienti personalizzati**.
1. Selezionare la casella di controllo nell'ambiente personalizzato dall'elenco.
1. Selezionare **Avanti**.
1. In Macchina virtuale selezionare **Standard_D2as_v4**.
1. Impostare **Numero di istanze** su **1**.
1. Selezionare **Avanti** e quindi di nuovo **Avanti**.
1. Seleziona **Crea**.

Attendere che il modello venga distribuito, possono essere necessari fino a 10 minuti. È possibile controllare lo stato in **Notifiche** o nella sezione endpoint di Studio di Azure Machine Learning.

### Testare l'endpoint del modello sottoposto a training

1. A sinistra selezionare **Endpoint**.
1. Selezionare **car-evaluation-endpoint**.
1. Selezionare **Test**, in**Dati di input per testare l'endpoint** incollare questo esempio JSON.

    ```json
    {
        "symboling": 2,
        "make": "mitsubishi",
        "fuel-type": "gas",
        "aspiration": "std",
        "num-of-doors": "two",
        "body-style": "hatchback",
        "drive-wheels": "fwd",
        "engine-location": "front",
        "wheel-base": 93.7,
        "length": 157.3,
        "width": 64.4,
        "height": 50.8,
        "curb-weight": 1944,
        "engine-type": "ohc",
        "num-of-cylinders": "four",
        "engine-size": 92,
        "fuel-system": "2bbl",
        "bore": 2.97,
        "stroke": 3.23,
        "compression-ratio": 9.4,
        "horsepower": 68.0,
        "peak-rpm": 5500.0,
        "city-mpg": 31,
        "highway-mpg": 38,
        "price": 0.0
    }
    ```

1. Selezionare **Testa**e verrà visualizzata una risposta:

    ```json
    {
        "predicted_price": 5852.823214312815
    }
    ```

1. Selezionare **Utilizza**.

    ![Screenshot che mostra come copiare l'endpoint REST e la chiave primaria.](../media/06-media/copy-rest-endpoint.png)
1. Copiare l'endpoint **REST**.
1. Copiare la **chiave primaria**.

### Integrare un modello di Azure Machine Learning con Azure AI Search

Successivamente, si crea un nuovo servizio di ricerca cognitiva e arricchire un indice usando un set di competenze personalizzato.

### Creare un file di test

1. Nel portale di Azure, selezionare [Gruppi di risorse](https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true).
1. Selezionare **aml-for-acs-enrichment**.

    ![Screenshot che mostra la selezione di un account di archiviazione nel portale di Azure.](../media/06-media/navigate-storage-account.png)
1. Selezionare l'account di archiviazione, ad esempio **amlforacsworks1440637584**.
1. Selezionare **Configurazione** in **Impostazioni**. Impostare quindi **Consenti accesso anonimo BLOB** su **Abilitato**.
1. Seleziona **Salva**.
1. In **Archiviazione dati** selezionare **Contenitori**. 
1. Creare un nuovo contenitore per archiviare i dati dell'indice, selezionare **+ Contenitore**.
1. Nel riquadro **Nuovo contenitore** in **Nome** immettere **search-data**.
1. In **Livello di accesso anonimo** selezionare **Contenitore (accesso in lettura anonimo per contenitori e BLOB)**.
1. Seleziona **Crea**.
1. Selezionare il contenitore **docs-to-search** creato.
1. In un editor di testo creare un documento JSON:

    ```json
    {
      "symboling": 0,
      "make": "toyota",
      "fueltype": "gas",
      "aspiration": "std",
      "numdoors": "four",
      "bodystyle": "wagon",
      "drivewheels": "fwd",
      "enginelocation": "front",
      "wheelbase": 95.7,
      "length": 169.7,
      "width": 63.6,
      "height": 59.1,
      "curbweight": 2280,
      "enginetype": "ohc",
      "numcylinders": "four",
      "enginesize": 92,
      "fuelsystem": "2bbl",
      "bore": 3.05,
      "stroke": 3.03,
      "compressionratio": 9.0,
      "horsepower": 62.0,
      "peakrpm": 4800.0,
      "citympg": 31,
      "highwaympg": 37,
      "price": 0
    }
    ```

    Salvare il documento nel computer come `test-car.json` estensione.
1. Nel portale selezionare **Carica**.
1. Nel riquadro **Carica BLOB** selezionare **Esplora file**, passare al percorso in cui è stato salvato il documento JSON e selezionarlo.
1. Selezionare **Carica**.

### Creare una risorsa di Azure AI Search

1. Nella home page del portale di Azure selezionare **+ Crea una risorsa**.
1. Cercare **Ricerca di intelligenza artificiale** di Azure e quindi selezionare **Ricerca di intelligenza artificiale di Azure**.
1. Seleziona **Crea**.
1. In **Gruppo di risorse** selezionare **aml-for-acs-enrichment**.
1. In Nome servizio immettere un nome univoco, ad esempio **acs-enriched-1440637584**.
1. Per **Località**, selezionare la stessa area usata in precedenza.
1. Selezionare **Rivedi e crea** e quindi **Crea**.
1. Attendere la distribuzione delle risorse e quindi selezionare **Vai alla risorsa**.
1. Selezionare **Importa dati**.
1. Nel riquadro **Connetti ai dati** selezionare **Archiviazione BLOB di Azure** per il campo **Origine dati**.
1. In **Nome origine dati**, immettere **import-docs**.
1. In **modalità analisi**, selezionare **JSON**.
1. In **Stringa di connessione** selezionare **Scegli una connessione esistente**.
1. Selezionare l'account di archiviazione su cui è stato caricato il file, ad esempio **amlforacsworks1440637584**.
1. Nel riquadro **Contenitori**, selezionare **docs-to-search**. 
1. Seleziona **Seleziona**.
1. Selezionare **Avanti: Aggiungi competenze cognitive (facoltativo)**.

### Aggiungere competenze cognitive

1. Espandere **Aggiungi arricchimenti**, quindi selezionare **Estrai nomi utenti**.
1. Selezionare **Avanti: Personalizza indice di destinazione**.
1. Selezionare **+ Aggiungi campo**, nel **nome campo** immettere **predicted_price** nella parte inferiore dell'elenco.
1. In **Tipo**, selezionare **Edm.Double** per la nuova voce.
1. Selezionare **Recuperabile** per tutti i campi.
1. Selezionare **Cercabile** per **crea**.
1. Seleziona **Successivo: Crea un indicizzatore**.
1. Selezionare **Invia**.

## Aggiungere la competenza AML al set di competenze

Si sostituiranno ora i nomi delle persone che si arricchiscono con il set di competenze personalizzato di Azure Machine Learning.

1. Nel riquadro Panoramica selezionare **Set di competenze** in **Gestione ricerca**.
1. In **Nome**selezionare **azureblob-skillset**.
1. Sostituire la definizione delle competenze per con questo codice JSON, ricordare di sostituire l'endpoint `EntityRecognitionSkill` copiato e i valori della chiave primaria:

    ```json
    "@odata.type": "#Microsoft.Skills.Custom.AmlSkill",
    "name": "AMLenricher",
    "description": "AML studio enrichment example",
    "context": "/document",
    "uri": "PASTE YOUR AML ENDPOINT HERE",
    "key": "PASTE YOUR PRIMARY KEY HERE",
    "resourceId": null,
    "region": null,
    "timeout": "PT30S",
    "degreeOfParallelism": 1,
    "inputs": [
      {
        "name": "symboling",
        "source": "/document/symboling"
      },
      {
        "name": "make",
        "source": "/document/make"
      },
      {
        "name": "fuel-type",
        "source": "/document/fueltype"
      },
      {
        "name": "aspiration",
        "source": "/document/aspiration"
      },
      {
        "name": "num-of-doors",
        "source": "/document/numdoors"
      },
      {
        "name": "body-style",
        "source": "/document/bodystyle"
      },
      {
        "name": "drive-wheels",
        "source": "/document/drivewheels"
      },
      {
        "name": "engine-location",
        "source": "/document/enginelocation"
      },
      {
        "name": "wheel-base",
        "source": "/document/wheelbase"
      },
      {
        "name": "length",
        "source": "/document/length"
      },
      {
        "name": "width",
        "source": "/document/width"
      },
      {
        "name": "height",
        "source": "/document/height"
      },
      {
        "name": "curb-weight",
        "source": "/document/curbweight"
      },
      {
        "name": "engine-type",
        "source": "/document/enginetype"
      },
      {
        "name": "num-of-cylinders",
        "source": "/document/numcylinders"
      },
      {
        "name": "engine-size",
        "source": "/document/enginesize"
      },
      {
        "name": "fuel-system",
        "source": "/document/fuelsystem"
      },
      {
        "name": "bore",
        "source": "/document/bore"
      },
      {
        "name": "stroke",
        "source": "/document/stroke"
      },
      {
        "name": "compression-ratio",
        "source": "/document/compressionratio"
      },
      {
        "name": "horsepower",
        "source": "/document/horsepower"
      },
      {
        "name": "peak-rpm",
        "source": "/document/peakrpm"
      },
      {
        "name": "city-mpg",
        "source": "/document/citympg"
      },
      {
        "name": "highway-mpg",
        "source": "/document/highwaympg"
      },
      {
        "name": "price",
        "source": "/document/price"
      }
    ],
    "outputs": [
      {
        "name": "predicted_price",
        "targetName": "predicted_price"
      }
    ]  
    ```

1. Seleziona **Salva**.

### Aggiornare i mapping dei campi di output

1. Nel riquadro **Panoramica** selezionare **Indicizzatori** e quindi selezionare **azureblob-indexer**.
1. Selezionare la scheda **Definizione indicizzatore (JSON)**, quindi modificare il valore **outputFieldMappings** in:

    ```json
    "outputFieldMappings": [
        {
          "sourceFieldName": "/document/predicted_price",
          "targetFieldName": "predicted_price"
        }
      ]
    ```

1. Seleziona **Salva**.
1. Selezionare **Reimposta** e quindi **Sì**.
1. Selezionare **Esegui** e quindi **Sì**.

## Testare l'arricchimento degli indici

Il set di competenze aggiornato aggiungerà ora un valore stimato al documento dell'auto di prova nell'indice. Per eseguire il test, seguire questa procedura.

1. Nel riquadro **Panoramica** del servizio di ricerca selezionare **Esplora ricerche** nella parte superiore del riquadro.
1. Seleziona **Cerca**.
1. Scorrere fino in fondo alla pagina.
    ![Screenshot che mostra il campo previsto relativo al prezzo dell'auto aggiunto ai risultati della ricerca.](../media/06-media/test-results-search-explorer.png)
Verrà visualizzato il campo popolato `predicted_price`.

## Eliminare le risorse dell'esercizio

Dopo aver completato l'esercizio, eliminare tutte le risorse non più necessarie. Eliminare le risorse di Azure:

1. Nel portale di Azure, selezionare **Gruppi di risorse**.
1. Selezionare il gruppo di risorse non necessario, quindi selezionare **Elimina gruppo di risorse**.
