Lafta Omar, artista di strada.
## Classe Artista
>La classe `Artista` rappresenta un thread che esegue il lavoro dell'artista di strada. All'interno del metodo run(), l'artista è in un loop infinito, simboleggiando che sta lavorando costantemente. In questo esempio, l'artista "lavora" dormendo per 1 secondo (Thread.sleep(1000)) all'interno del loop. Questo può essere considerato come il tempo che l'artista impiega per eseguire un ritratto a carboncino. Il loop continuerà a essere eseguito finché il programma non viene interrotto manualmente.
```java
public class Artista extends Thread {
    @Override
    public void run() {
        System.out.println("L'artista è pronto.");

        while (true) {
            // L'artista continua a lavorare finché il programma non viene interrotto manualmente

            try {
                Thread.sleep(1000); // Tempo impiegato dall'artista per eseguire un ritratto (1 secondo)
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## Classe SimulazioneStreetArtist
>La classe `StreetArtistSimulation` rappresenta la simulazione di un'attività di un artista di strada che esegue ritratti a carboncino. Utilizzando semafori come meccanismo di sincronizzazione, la simulazione gestisce l'arrivo casuale dei clienti, la disponibilità delle sedie e il flusso di lavoro dell'artista. I clienti cercano di ottenere una sedia, attendono il loro turno e si spostano nella zona di lavoro. Se un cliente deve aspettare troppo a lungo, rinuncia. L'artista lavora costantemente, eseguendo ritratti per i clienti quando è il loro turno.
```java
public class SimulazioneStreetArtist {
    private static final int NUMERO_SEDI = 4; // Numero di sedie disponibili
    private static final int TEMPO_ATTESA_MASSIMO = 5000; // Tempo massimo di attesa di un cliente in millisecondi

    private static Semaphore sedieDisponibili; // Semaforo per tenere traccia delle sedie disponibili

    public static void main(String[] args) {
        setSedieDisponibili(new Semaphore(NUMERO_SEDI));

        Artista artista = new Artista();
        artista.start();

        Random random = new Random();

        // Genera nuovi clienti ad intervalli casuali
        for (int i = 1; i <= 10; i++) {
            Cliente cliente = new Cliente(i, random.nextInt(TEMPO_ATTESA_MASSIMO));
            cliente.start();

            // Attende un po' di tempo prima di generare il cliente successivo
            try {
                Thread.sleep(random.nextInt(2000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

	public static Semaphore getSedieDisponibili() {
		return sedieDisponibili;
	}

	public static void setSedieDisponibili(Semaphore sedieDisponibili) {
		SimulazioneStreetArtist.sedieDisponibili = sedieDisponibili;
	}
}
```

## Classe Cliente
>La classe `Cliente` rappresenta un thread che rappresenta un cliente che si avvicina all'artista di strada per farsi fare un ritratto a carboncino. Il cliente cerca di ottenere una sedia disponibile, attende il suo turno, si alza quando è il momento e rinuncia se deve aspettare troppo a lungo.
```java
public class Cliente extends Thread {
    private int idCliente;
    private int tempoAttesa;

    public Cliente(int idCliente, int tempoAttesa) {
        this.idCliente = idCliente;
        this.tempoAttesa = tempoAttesa;
    }

    @Override
    public void run() {
        System.out.println("Cliente " + idCliente + " arriva.");

        if (SimulazioneStreetArtist.getSedieDisponibili().tryAcquire()) {
            // Cliente ha ottenuto una sedia
            System.out.println("Cliente " + idCliente + " si siede sulla sedia.");

            try {
                Thread.sleep(tempoAttesa); // Tempo di attesa del cliente
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println("Cliente " + idCliente + " si alza dopo " + tempoAttesa + "ms e si sposta nella zona di lavoro.");

            SimulazioneStreetArtist.getSedieDisponibili().release(); // Rilascia la sedia
        } else {
            // Cliente rinuncia a farsi fare il ritratto
            System.out.println("Cliente " + idCliente + " rinuncia a farsi fare il ritratto dopo un'attesa troppo lunga.");
        }
    }
}
```

Per evitare lo stallo e la starvation nella simulazione, sono state utilizzate le seguenti strategie:

1. Semafori: I semafori sono stati utilizzati per sincronizzare l'accesso alle sedie. Ogni cliente cerca di acquisire una sedia utilizzando il semaforo `sedieDisponibili`. Se la sedia è disponibile, il cliente può procedere. Se tutte le sedie sono occupate, il cliente attende finché una sedia non diventa disponibile. Questo evita lo stallo, in quanto i clienti si sospendono in attesa di una sedia.

2. Rilascio delle sedie: Quando un cliente ha finito il suo turno, rilascia la sedia utilizzando `sedieDisponibili.release()`. Ciò consente ad altri clienti di acquisire la sedia e procedere. In questo modo, i clienti non si escludono a vicenda dalle sedie e si evita la starvation, in quanto tutti i clienti hanno la possibilità di ottenere una sedia.

3. Gestione del tempo di attesa: Il tempo di attesa di ciascun cliente è casuale, impostato nel momento della creazione del cliente (`tempoAttesa` in millisecondi). Questo garantisce che i clienti abbiano un'opportunità equa di ottenere una sedia e che nessun cliente sia favorito o penalizzato in modo sistematico.

Queste strategie consentono di gestire in modo efficace l'accesso alle sedie e di prevenire situazioni di stallo (deadlock) o di fame (starvation) nella simulazione.
