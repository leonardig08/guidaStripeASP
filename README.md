# Guida all'Implementazione di Stripe su ASP.NET Core 🚀

Questa guida ti mostrerà passo dopo passo come integrare il sistema di pagamento **Stripe** nella tua applicazione **ASP.NET Core**. Prima di iniziare, assicurati di avere già un'applicazione web con un negozio, un sistema di checkout e un sistema di gestione degli account utente già funzionanti.

---

## 1. Configurazione dell'Account Stripe 🔑

Il primo passo è creare un account Stripe in modalità "sandbox" per i test.

1.  **Registrati** sul [sito ufficiale di Stripe](https://stripe.com/). Puoi accedere rapidamente utilizzando il tuo account Google.
![immagine](img/1.jpg)
2.  Completa il processo di creazione dell'account.
![immagine](img/2.jpg)
3.  Una volta completato, clicca su **"Vai alla sandbox"** per attivare l'ambiente di test. !
![immagine](img/3.jpg)
4.  Nel menu di navigazione, seleziona **"Sviluppatori"** e poi **"Chiavi API"**.
![immagine](img/4.jpg)

---

## 2. Inserimento delle Chiavi API nell'Applicazione 📝

Ora che hai le tue chiavi, le utilizzeremo per connettere la tua applicazione a Stripe.

1.  Copia le due chiavi (la **Chiave Segreta** e la **Chiave Pubblicabile**) dalla console di Stripe.
![immagine](img/5.jpg)
2.  Nel tuo progetto ASP.NET Core, apri il file `appsettings.json` e aggiungi la seguente sezione per memorizzare le chiavi:

    ```json
    "Stripe": {
      "SecretKey": "sk_test_...", // Incolla qui la tua Chiave Segreta
      "PublishableKey": "pk_test_..." // Incolla qui la tua Chiave Pubblicabile
    }
    ```

Ora la tua applicazione è pronta per interagire con l'API di Stripe!

---

## 3. Installazione della Libreria Stripe 📦

Per poter utilizzare l'API di Stripe nel tuo codice, devi installare la libreria ufficiale.

1.  In Visual Studio, vai su **Strumenti** → **Gestione Pacchetti NuGet** → **Gestisci pacchetti NuGet per la soluzione...**.
![immagine](img/6.jpg)
2.  Cerca il pacchetto **`Stripe.net`** e installalo nel tuo progetto.
![immagine](img/7.jpg)

---

## 4. Configurazione del Controller di Checkout 🛠️

Iniziamo a scrivere il codice per gestire il processo di pagamento.

1.  Nel tuo controller di **Checkout**, importa le librerie necessarie:

    ```csharp
    using Stripe;
    using Stripe.Checkout;
    ```

2.  Aggiungi le seguenti righe all'inizio del tuo controller. Assicurati di sostituire `ApexVolleyContext` con il tuo `DbContext`.

    ```csharp
    private readonly ApexVolleyContext _context;
    private readonly IConfiguration _config;

    public CheckoutController(ApexVolleyContext context, IConfiguration config)
    {
      _context = context;
      _config = config;
      // Inizializza la chiave segreta di Stripe
      StripeConfiguration.ApiKey = _config["Stripe:SecretKey"];
    }
    ```

---

## 5. Creazione di una Sessione di Pagamento Stripe 🛒

Stripe Checkout reindirizza i clienti su una pagina di pagamento sicura ospitata da Stripe, semplificando il processo.

1.  Crea un'istanza di `SessionCreateOptions` per definire i dettagli del pagamento. Questo include il prezzo, i nomi dei prodotti e le quantità.

    ```csharp
    var options = new SessionCreateOptions
    {
      PaymentMethodTypes = new List<string> { "card" },
      LineItems = cart.Select(c => new SessionLineItemOptions
      {
        PriceData = new SessionLineItemPriceDataOptions
        {
          UnitAmount = (long)(c.Product.Price * 100), // Prezzo in centesimi
          Currency = "eur",
          ProductData = new SessionLineItemPriceDataProductDataOptions
          {
            Name = c.Product.Name
          }
        },
        Quantity = c.Quantity
      }).ToList(),
      Mode = "payment",
      // URL di reindirizzamento dopo il successo o l'annullamento
      SuccessUrl = Url.Action("Success", "Checkout", new { orderId = order.Id }, Request.Scheme),
      CancelUrl = Url.Action("Index", "Checkout", null, Request.Scheme),
      ClientReferenceId = order.Id.ToString()
    };
    ```

2.  Crea la sessione e reindirizza l'utente:

    ```csharp
    var service = new SessionService();
    var session = service.Create(options);

    if (session == null || string.IsNullOrEmpty(session.Url))
    {
      TempData["ErrorMessage"] = "Errore nella creazione del pagamento.";
      return RedirectToAction("Index");
    }

    return Redirect(session.Url);
    ```

---

## 6. Gestione del Successo e dell'Annullamento del Pagamento ✅

Dopo il pagamento, Stripe reindirizza l'utente agli URL che hai specificato.

1.  Crea un'azione per gestire il successo del pagamento. In questo esempio, l'utente viene reindirizzato alla pagina dei dettagli dell'ordine.

    ```csharp
    [HttpGet]
    public IActionResult Success(int orderId)
    {
      TempData["SuccessMessage"] = "Pagamento completato! Il tuo ordine verrà processato.";
      return RedirectToAction("Details", "Orders", new { id = orderId });
    }
    ```

---

## 7. Verifica dei Pagamenti con Stripe-CLI e Webhook 🕸️

Per verificare che il pagamento sia andato a buon fine, dovrai usare un **webhook**. In ambiente di sviluppo, useremo **Stripe-CLI** per simulare la comunicazione con Stripe. !

1.  **Installa Stripe-CLI**: vai alla [pagina GitHub per il download di Stripe-CLI](https://github.com/stripe/stripe-cli/releases) e scarica la versione appropriata per il tuo sistema operativo (ad es., `windows_x86_64`).
![immagine](img/8.jpg)
2.  Estrai il file `.zip` in una cartella a tua scelta.
![immagine](img/9.jpg)
3.  Cerca e apri lo strumento per modificare le **variabili d'ambiente** del sistema.
![immagine](img/10.jpg)
4.  Aggiungi il percorso dell'eseguibile di Stripe-CLI alla variabile di ambiente `Path`. !
![immagine](img/11.jpg)

Spero che questa guida ti sia utile per integrare i pagamenti Stripe nella tua applicazione!
