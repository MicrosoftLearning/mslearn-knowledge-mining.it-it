---
lab:
  title: Configurare il classificatore semantico
---

# Configurare il classificatore semantico

> **Nota:** per completare questo lab è necessaria una [sottoscrizione di Azure](https://azure.microsoft.com/free?azure-portal=true) in cui si ha accesso amministrativo. Questo esercizio richiede anche il servizio **Azure AI Search** con un piano fatturabile.

In questo esercizio si aggiungerà a un indice un classificatore semantico e lo si userà per una query.

## Abilitare il classificatore semantico

1. Apri il portale di Azure e esegui l’accesso.
1. Seleziona **Tutte le risorse** e seleziona il servizio di ricerca.
1. Nel riquadro di spostamento, selezionare **Classificatore semantico**.
1. In **Disponibilità**, nell'opzione **Gratuito** selezionare **Seleziona piano**.

![Screenshot della finestra di dialogo del classificatore semantico.](../media/semantic-search/semanticsearch.png)

## Importare un indice di esempio

1. Tornare alla pagina **Panoramica** del servizio di ricerca.
1. Selezionare **Importa dati**.

    ![Screenshot del pulsante Importa dati.](../media/semantic-search/importdata.png)

1. In **Origine dati** selezionare **Esempi**.
1. Selezionare **hotels-sample**, quindi selezionare **Avanti: Aggiungi competenze cognitive (facoltativo)**.
1. Selezionare **Passa a: Personalizza indice di destinazione**.
1. Seleziona **Successivo: Crea un indicizzatore**.
1. Selezionare **Invia**.

## Configura la classificazione semantica

Dopo aver abilitato un indice di ricerca e un classificatore semantico, è possibile configurare la classificazione semantica. È necessario un client di ricerca che supporti le API di anteprima nella richiesta di query. È possibile usare Esplora ricerche nel portale di Azure, l'app Postman, l’SDK Azure per .NET o l’SDK Azure per Python. In questo esercizio si userà Esplora ricerche nel portale di Azure.

Per configurare la classificazione semantica, segui questa procedura:

1. Nella barra di spostamento, in **Gestione ricerca**, seleziona **Indici**.

    ![Screenshot del pulsante Indici.](../media/semantic-search/indexes.png)

1. Seleziona l'indice.
1. Seleziona **Configurazioni semantiche** e seleziona **Aggiungi configurazione semantica**.
1. In **Nome** digitare **hotels-conf**.
1. Nel **campo Titolo** selezionare **HotelName**.
1. Nei **campi Contenuto** in **Nome campo** selezionare **Descrizione**.
1. Ripetere il passaggio precedente per i campi seguenti:
    - **Categoria**
    - **Indirizzo/Città**
1. In **Campi parola chiave**, in **Nome campo** selezionare **Tag**.
1. Seleziona **Salva**.
1. Nella pagina dell'indice, seleziona **Salva**.
1. Selezionare **Esplora ricerche**.
1. Selezionare **Visualizza** e selezionare **Visualizzazione JSON**.
1. Nell'editor di query JSON digitare il testo seguente:

    ```json
        {
         "queryType": "semantic",
         "queryLanguage" : "en-us",
         "search": "all hotels near the water" , 
         "semanticConfiguration": "hotels-conf" , 
         "searchFields": "",
         "speller": "lexicon" , 
         "answers": "extractive|count-3",
         "count": true
        }
    ```

1. Seleziona **Cerca**.
1. Esaminare i risultati della query.

## Eliminazione

Se non è più necessario il servizio Azure AI Search, è necessario eliminare la risorsa dalla sottoscrizione di Azure per ridurre i costi.

>**Nota** L'eliminazione del servizio Azure AI Search garantisce che la sottoscrizione non venga addebitata per le risorse. Verrà tuttavia addebitato un importo ridotto per l'archiviazione dei dati, fintanto che lo spazio di archiviazione è presente nella sottoscrizione. Se l'esplorazione del servizio Ricerca cognitiva è stata completata, è possibile eliminare il servizio Ricerca cognitiva e le risorse associate. Tuttavia, se si prevede di completare qualsiasi altro lab in questa serie, sarà necessario ricrearla.
> Per eliminare le risorse:
> 1. Nel [portale di Azure](https://portal.azure.com?azure-portal=true ), nella pagina **Gruppi di risorse**, aprire il gruppo di risorse specificato durante la creazione del servizio Ricerca cognitiva.
> 1. Fare clic su **Elimina gruppo di risorse**, digitare il nome del gruppo di risorse per confermare che si vuole eliminarlo e quindi selezionare **Elimina**.
