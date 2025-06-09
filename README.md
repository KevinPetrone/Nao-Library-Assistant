UNIVERSITÀ DEGLI STUDI DI PALERMO  
FACOLTÀ DI INGEGNERIA

RELAZIONE TECNICA  
SISTEMA NAO LIBRARY ASSISTANT  
CON INTEGRAZIONE ROS2-WEBOTS

**Sviluppatori:** Petrone Kevin, Di Peri Marta, Cirincione Marika  
**Matricola:** 0774409, 0774023, 0772935  
**Docente:** Seidita Valeria

Progetto per esame di  
programmazione dei robot  
Scenario 3

**Corso:** Ing. Robotica  
**Anno Accademico:** 2024/2025  
**Data:** Giugno 2025  
  
**Tecnologie:**  
ROS2 Humble • Webots R2025a • Python 3.10

INDICE

1\. INTRODUZIONE

2\. BASE DI CONOSCENZA, GESTIONE DATABASE

2\. ARCHITETTURA DEL SISTEMA

3\. FLUSSO DI COMUNICAZIONE

4\. STRUTTURA DEL PROGETTO

5\. SISTEMA DI NAVIGAZIONE

6\. INSTALLAZIONE E CONFIGURAZIONE

7\. INTERFACCIA UTENTE

9\. RISOLUZIONE PROBLEMI

10\. CONCLUSIONI

1\. INTRODUZIONE
================

Il progetto **NAO Library Assistant** rappresenta un sistema robotico avanzato che integra ROS2 (Robot Operating System 2) con il simulatore Webots R2025a per controllare un robot NAO virtuale in un ambiente bibliotecario simulato.

Il sistema permette all'utente di interagire tramite comandi testuali per richiedere libri specifici, mentre il robot NAO risponde alle richieste e naviga autonomamente tra gli scaffali della biblioteca per accompagnare l'utente allo scaffale del libro richiesto, dimostrando un'applicazione pratica della robotica di servizio.

1.1 Obiettivi del Sistema
-------------------------

*   Fornire informazioni dettagliate sui libri presenti nella biblioteca
*   Aggiornare e comunicare lo stato di disponibilità dei volumi
*   Accompagnare virtualmente l’utente fino allo scaffale che contiene il libro desiderato

1.2 Comportamento atteso del robot
----------------------------------

Il comportamento del robot si sviluppa in modo strutturato: saluta l’utente, propone una lista di opzioni numerate da 1 a 9 (relative a informazioni su autore, genere, anno, disponibilità, trama, posizione, accompagnamento, restituzione e gestione), interpreta la scelta e risponde con l’audio appropriato. Se viene scelta l’opzione 7, il robot si muove nello spazio simulato fino allo scaffale selezionato, parlando tramite audio durante il tragitto, e infine ritorna alla posizione iniziale.

1.3 Dominio del sistema
-----------------------

Il progetto si colloca in una biblioteca semistrutturata, dove il robot NAO assiste gli utenti nella ricerca e consultazione dei libri. L’ambiente è stato ricreato fedelmente partendo da una scansione reale dell’aula, successivamente modellata con il software Fusion 360. Gli oggetti sono stati poi esportati in formato OBJ e importati in Webots come solidi CADShape, per garantire una simulazione coerente e realistica.

![Scansione dell'aula](https://raw.githubusercontent.com/KevinPetrone/Nao-Library-Assistant/main/images/aula_foto.png)

Figura 1 – Rappresentazione dell'aula scansionata e ricostruita

2\. BASE DI CONOSCENZA, GESTIONE DATABASE
=========================================

2.1 Descrizione Della Base Di Conoscenza
----------------------------------------

Il sistema robotico impiegato nel progetto “Servizio in biblioteca” si basa su una base di conoscenza strutturata in formato JSON, progettata per rappresentare in modo dettagliato l’insieme dei libri presenti nella biblioteca simulata. Ogni libro è descritto come un oggetto JSON autonomo, composto da una serie di attributi chiave che ne delineano sia le caratteristiche bibliografiche sia le informazioni logistiche. All’interno del progetto, questa base di conoscenza costituisce il nucleo informativo del sistema: consente al robot di comprendere e rispondere in maniera coerente e contestualizzata alle richieste dell’utente, fungendo da interfaccia tra l’interazione vocale e i dati memorizzati. Dal punto di vista concettuale, la base di conoscenza può essere interpretata come una sorta di ontologia descrittiva, in cui le relazioni tra entità (ad esempio: “il libro ha autore”, “il libro appartiene a un genere”, “il libro si trova in una posizione”) sono implicitamente definite tramite i campi del JSON.

2.2 Struttura Database JSON
---------------------------

La base di conoscenza è costituito da un database `database_libri.json` che utilizza una struttura JSON ottimizzata per ricerche rapide:

{ "Il Signore degli Anelli": { "titolo": "Il Signore degli Anelli", "autore": "J.R.R. Tolkien", "genere": "Fantasy epico", "anno": 1954, "disponibile": true, "carta\_biblioteca": true, "trama": "...", "location": {...} }, "Dune": { "titolo": "Dune", "autore": "Frank Herbert", "genere": "Fantascienza", "anno": 1965, "disponibile": true, "carta\_biblioteca": true, "trama": "...", "location": {...} }, "Il nome della Rosa":{ "titolo": "Il Nome della Rosa", "autore": "Umberto Eco", "genere": "Gialli storico", "anno": 1980, "disponibile": true, "carta\_biblioteca": true, "trama": "...", "location": {...} } }

2.3 Classe Database Manager
---------------------------

class LibraryDatabase: def \_\_init\_\_(self, database\_path): self.database\_path = database\_path self.books\_data = self.load\_database() def search\_book(self, query): """Ricerca libro per titolo, autore o genere""" query\_lower = query.lower() for book in self.books\_data\['libri'\]: if (query\_lower in book\['titolo'\].lower() or query\_lower in book\['autore'\].lower() or query\_lower in book\['genere'\].lower()): return book return None def get\_books\_by\_shelf(self, shelf\_row): """Ottiene tutti i libri di uno scaffale specifico""" return \[book for book in self.books\_data\['libri'\] if book\['shelf\_row'\] == shelf\_row and book\['disponibile'\]\]

2.4 Grafo Ontologia OWL
-----------------------

Le frecce colorate rappresentano relazioni semantiche (es. “ha autore”, “è di genere”, “è collocato in”).  
  
I diamanti e cerchi indicano classi e istanze distinte.  
  
Alcuni nodi (come “JKRowling” o “Fantasy”) rappresentano esempi di individui usati a scopo illustrativo.

![Scansione dell'aula](https://raw.githubusercontent.com/KevinPetrone/Nao-Library-Assistant/main/images/mappa_owl.jpg)

Figura 2 – Rappresentazione Ontograf

3\. ARCHITETTURA DEL SISTEMA
============================

3.1 Schema Generale dell'Architettura
-------------------------------------

┌─────────────────────────────────────────────────────────────────┐ │ SISTEMA NAO LIBRARY ASSISTANT │ ├─────────────────────────────────────────────────────────────────┤ │ │ │ ┌─────────────────┐ ┌──────────────────┐ ┌─────────────┐ │ │ │ INTERFACCIA │ │ ELABORAZIONE │ │ SIMULAZIONE │ │ │ │ UTENTE │ │ LOGICA │ │ FISICA │ │ │ │ │ │ │ │ │ │ │ │ ┌─────────────┐ │ │ ┌──────────────┐ │ │ ┌─────────┐ │ │ │ │ │ Keyboard │ │ │ │ Library │ │ │ │ Webots │ │ │ │ │ │ Input Node │ │───>| | Assistant │ │───>| | NAO │ │ │ │ │ │ │ │ │ │ │ │ │ │ Robot │ │ │ │ │ └─────────────┘ │ │ └──────────────┘ │ │ └─────────┘ │ │ │ │ │ │ ┌──────────────┐ │ │ ┌─────────┐ │ │ │ │ │ │ │ Database │ │ │ │ Library │ │ │ │ │ │ │ │ Manager │ │ │ │ World │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ └──────────────┘ │ │ └─────────┘ │ │ │ └─────────────────┘ └──────────────────┘ └─────────────┘ │ │ │ └─────────────────────────────────────────────────────────────────┘

Figura 3: Architettura generale del sistema NAO Library Assistant

3.2 Componenti del Sistema
--------------------------

### 3.2.1 Nodi ROS2

Nodo

Funzione

Tipo

**keyboard\_input\_node**

Gestisce l'input utente da tastiera e pubblica comandi

Publisher

**library\_assistant**

Elabora richieste utente, gestisce database e logica di business

Subscriber/Publisher

**nao\_controller\_node**

Bridge tra ROS2 e controllo robot Webots

Subscriber

### 3.2.2 Topics di Comunicazione

Topic

Tipo Messaggio

Descrizione

**user\_input**

String

Trasporta comandi utente dal nodo di input

**nao\_speech**

String

Comandi vocali e messaggi per il robot NAO

**nao\_movement**

Int32

Comandi di movimento specifici per navigazione

4\. FLUSSO DI COMUNICAZIONE
===========================

4.1 Diagramma di Flusso Dati
----------------------------

┌─────────────┐ user\_input ┌─────────────────┐ │ Keyboard │────────────────────▶│ Library │ │ Input │ (String) │ Assistant │ │ Node │ │ │ └─────────────┘ └─────────────────┘ │ │ nao\_movement │ (Int32) ▼ ┌─────────────────┐ │ NAO Controller │ │ Node │ │ │ └─────────────────┘ │ │ Robot Commands ▼ ┌─────────────────┐ │ Webots NAO │ │ Simulation │ │ │ └─────────────────┘

Figura 2: Flusso di comunicazione tra i componenti del sistema

4.2 Sequenza di Operazioni
--------------------------

1.  **Input Utente**: L'utente digita un comando specifico nel terminale
2.  **Pubblicazione**: Il nodo keyboard\_input\_node pubblica il comando sul topic 'user\_input'
3.  **Elaborazione**: Il nodo library\_assistant riceve e processa la richiesta utente
4.  **Database Query**: Viene effettuata una ricerca nel file database\_libri.json
5.  **Comando Movimento**: Viene pubblicato sul topic 'nao\_movement' il codice shelf\_row
6.  **Controllo Robot**: Il nodo nao\_controller\_node riceve il comando e attiva il percorso
7.  **Simulazione**: Webots esegue fisicamente i movimenti del robot NAO virtuale

**✅ Vantaggi dell'Architettura:**  
L'utilizzo del pattern Publisher-Subscriber di ROS2 garantisce un disaccoppiamento completo tra i componenti, permettendo modularità e facilità di manutenzione del sistema.

5\. STRUTTURA DEL PROGETTO
==========================

5.1 Organizzazione Directory
----------------------------

nao\_library\_ws/ ├── src/nao\_library\_assistant/ │ ├── nao\_library\_assistant/ # Moduli Python ROS2 │ │ ├── \_\_init\_\_.py │ │ ├── keyboard\_input\_node.py # Gestione input tastiera │ │ ├── library\_assistant.py # Logica principale sistema │ │ ├── library\_database.py # Interfaccia database │ │ └── nao\_controller\_node.py # Controller robot Webots │ ├── controllers/go\_library/ # Controller Webots │ │ └── go\_library.py # Bridge ROS2-Webots │ ├── data/ │ │ ├── database\_libri.json # Database catalogo biblioteca │ │ └── password.json # password riservata al personale │ ├── worlds/ │ │ ├── esame\_Nao.wbt # Mondo Webots biblioteca │ │ └── oggetti.obj # Oggetti obj e mtl per l'ambiente │ ├── motions/ # File movimenti preregistrati │ │ ├── Forwards50.motion │ │ ├── TurnLeft60.motion │ │ ├── TurnRight60.motion │ │ └── HandWave.motion │ ├── launch/ │ │ └── nao\_library.launch.py # File lancio sistema │ ├── Audio/ # File audio sistema │ ├── package.xml # Configurazione pacchetto │ ├── setup.py # Setup Python │ └── start\_nao\_library.sh # Script avvio rapido └── install/ # Directory build (generata automaticamente) └── nao\_library\_assistant/ ├── lib/nao\_library\_assistant/ # Eseguibili compilati │ ├── keyboard\_input\_node │ ├── library\_assistant │ └── nao\_controller\_node └── share/nao\_library\_assistant/ # Risorse condivise ├── launch/ ├── worlds/ ├── controllers/ └── data/

5.2 Importanza della Directory Install
--------------------------------------

**⚠️ ATTENZIONE CRITICA:**  
La directory `install/` è generata automaticamente dal comando `colcon build` e contiene:

*   **Eseguibili compilati**: Versioni ottimizzate pronte per l'esecuzione
*   **Moduli Python installati**: Importabili direttamente dal sistema ROS2
*   **Risorse condivise**: Launch files, mondi Webots, database e configurazioni

**Senza questa directory, il sistema non può funzionare!**

5.3 File di Configurazione Principali
-------------------------------------

File

Scopo

Formato

package.xml

Metadati pacchetto ROS2, dipendenze

XML

setup.py

Configurazione installazione Python

Python

database\_libri.json

Catalogo biblioteca con mappatura scaffali

JSON

esame\_Nao.wbt

Mondo 3D Webots con ambiente biblioteca

Webots World

6\. SISTEMA DI NAVIGAZIONE
==========================

6.1 Mappatura Biblioteca
------------------------

Il robot utilizza un sistema di **percorsi predefiniti** ottimizzati per raggiungere diverse sezioni della biblioteca. Ogni scaffale è associato a una sequenza specifica di movimenti:

self.shelf\_row\_paths = { 1: \[ # Scaffale Scienza e Cucina {"action": "FORWARD", "distance": 1.0}, {"action": "TURN\_LEFT", "angle": 45}, {"action": "FORWARD", "distance": 6.5}, {"action": "WAIT", "time": 10}, {"action": "RETURN\_HOME"} \], 2: \[ # Scaffale Bambini Ragazzi e Fantasy Fantascienza {"action": "FORWARD", "distance": 1.0}, {"action": "TURN\_RIGHT", "angle": 100}, {"action": "FORWARD", "distance": 1.57}, {"action": "WAIT", "time": 10}, {"action": "RETURN\_HOME"} \], 3: \[ # Scaffale Gialli e Narrativa {"action": "FORWARD", "distance": 2.0}, {"action": "TURN\_LEFT", "angle": 90}, {"action": "FORWARD", "distance": 3.0}, {"action": "WAIT", "time": 10}, {"action": "RETURN\_HOME"} \] }

**✅ Vantaggi del Sistema di Navigazione:**

*   **Affidabilità**: Percorsi testati e ottimizzati per ogni destinazione
*   **Prevedibilità**: Comportamento robot sempre coerente e ripetibile
*   **Efficienza**: Movimenti diretti senza calcoli real-time complessi
*   **Manutenibilità**: Facile modifica e aggiunta di nuovi percorsi

7\. INSTALLAZIONE E CONFIGURAZIONE
==================================

7.1 Prerequisiti Software
-------------------------

**⚠️ IMPORTANTE:** Utilizzare esclusivamente **Webots R2025a** (non R2023b o versioni precedenti) per garantire piena compatibilità con il sistema.

Software

Versione Richiesta

Comando Verifica

ROS2 Humble

Humble Hawksbill

`ros2 --version`

Webots

R2025a

`webots --version`

Python

3.10+

`python3 --version`

Colcon

Latest

`colcon --help`

7.2 Procedura di Installazione Completa
---------------------------------------

### Step 1: Preparazione su PC Sviluppo

\# Verifica prerequisiti sistema echo $ROS\_DISTRO # Deve mostrare ROS2 Humble webots --version # Deve mostrare Webots R2025a python3 --version # Deve essere 3.10 o superiore colcon --help # Verifica build tool

### Step 2: Creazione Archivio per Trasferimento

\# Creare archivio completo incluso codice sorgente cd ~/ tar -czf nao\_library\_complete.tar.gz nao\_library\_ws/ # Verifica dimensione archivio (dovrebbe essere circa 50-100MB) ls -lh nao\_library\_complete.tar.gz

### Step 3: Installazione su PC Destinazione

\# Trasferire archivio al PC di destinazione scp nao\_library\_complete.tar.gz utente@pc-destinazione:~/ # Accedere al PC destinazione ssh utente@pc-destinazione # Scompattare archivio cd ~/ tar -xzf nao\_library\_complete.tar.gz # Configurare ambiente ROS2 source /opt/ros/humble/setup.bash # REBUILD OBBLIGATORIO su ogni nuovo PC cd nao\_library\_ws/ colcon build --packages-select nao\_library\_assistant # Configurare workspace locale source install/setup.bash

7.3 Verifica Installazione
--------------------------

\# Verificare creazione eseguibili ls install/lib/nao\_library\_assistant/ # Output atteso: # keyboard\_input\_node library\_assistant nao\_controller\_node # Verificare risorse condivise ls install/share/nao\_library\_assistant/ # Output atteso: # controllers/ data/ launch/ worlds/ # Test rapido sistema ros2 pkg list | grep nao\_library\_assistant # Deve mostrare: nao\_library\_assistant

**✅ Controllo Installazione Riuscita:**  
Se tutti i comandi precedenti producono l'output atteso, l'installazione è completata con successo e il sistema è pronto per l'utilizzo.

8\. INTERFACCIA UTENTE
======================

8.1 Avvio del Sistema
---------------------

\# Terminal 1: Avvio sistema principale cd ~/nao\_library\_ws/src/nao\_library\_assistant chmod +x start\_nao\_library.sh ./start\_nao\_library.sh # Terminal 2: Avvio interfaccia utente (in parallelo) cd ~/nao\_library\_ws source /opt/ros/humble/setup.bash source install/setup.bash ros2 run nao\_library\_assistant keyboard\_input\_node

8.2 Keyboard Input Node
-----------------------

Il modulo `keyboard_input_node.py` gestisce l'interazione utente attraverso un'interfaccia a riga di comando intuitiva:

class KeyboardInputNode(Node): def \_\_init\_\_(self): super().\_\_init\_\_('keyboard\_input\_node') self.user\_input\_pub = self.create\_publisher(String, 'user\_input', 10) def keyboard\_input\_loop(self): """Loop principale per input utente""" print("=== NAO LIBRARY ASSISTANT ===") print("Comandi disponibili:") print("1-9: Menù dei comandi") print("Nome libro: Ricerca diretta nel catalogo") print("CTRL+C: Uscita dal sistema") print("=" \* 30) while rclpy.ok(): try: current\_input = input("📚 Comando > ") if current\_input.strip(): self.publish\_user\_input(current\_input.strip()) except KeyboardInterrupt: print("\\n👋 Uscita dal sistema...") break

8.3 Comandi Disponibili
-----------------------

Comando

Azione

Esempio

1-9

Menù dei comandi

`5` → Leggi la TRAMA di un libro

Nome libro

Ricerca nel database per titolo

`Il Signore degli Anelli`

CTRL+C

Uscita sicura dal sistema

Terminazione controllata

Il comando 1-9, offre 9 richieste da input da tastiera, tra cui:

Tasto

Richiesta

1️⃣

Cerca per AUTORE

2️⃣

Cerca per GENERE

3️⃣

Cerca per ANNO

4️⃣

Verifica DISPONIBILITÀ di un libro

5️⃣

Leggi la TRAMA di un libro

6️⃣

Trova la POSIZIONE di un libro

7️⃣

Voglio essere ACCOMPAGNATO a un libro specifico

8️⃣

RESTITUISCI un libro

9️⃣

GESTISCI libri (Aggiungi/Modifica/Elimina) "gestita da password"

8.4 Interfaccia Feedback
------------------------

**🖥️ Messaggi di Sistema:**

*   **Ricerca riuscita**: "📖 Libro trovato: \[Titolo\] - Scaffale \[N\]"
*   **Navigazione**: "🤖 NAO si sta dirigendo verso lo scaffale \[N\]"
*   **Completamento**: "✅ Missione completata - Scaffale raggiunto"
*   **Errore**: "❌ Libro non trovato nel catalogo"

9\. RISOLUZIONE PROBLEMI COMUNI
===============================

9.1 Errori di Build
-------------------

\# Problema: "Package not found during build" # Soluzione: Pulizia completa e rebuild rm -rf build/ install/ log/ colcon build --packages-select nao\_library\_assistant source install/setup.bash # Problema: "Permission denied" # Soluzione: Correzione permessi chmod +x src/nao\_library\_assistant/start\_nao\_library.sh sudo chown -R $USER:$USER nao\_library\_ws/

9.2 Errori Webots
-----------------

\# Problema: "Webots not found in PATH" # Soluzione: Aggiunta al PATH di sistema export PATH=$PATH:/usr/local/webots/bin echo 'export PATH=$PATH:/usr/local/webots/bin' >> ~/.bashrc # Problema: "World file not loading" # Soluzione: Verifica percorsi relativi ls install/share/nao\_library\_assistant/worlds/esame\_Nao.wbt

9.3 Errori ROS2
---------------

\# Problema: "Node not found" # Soluzione: Verifica sourcing ambiente source /opt/ros/humble/setup.bash source install/setup.bash ros2 pkg list | grep nao\_library\_assistant # Problema: "Topics not publishing" # Soluzione: Debug comunicazione ros2 topic list ros2 topic echo /user\_input ros2 node list

9.4 Errori di Dipendenze
------------------------

\# Problema: "Missing Python dependencies" # Soluzione: Installazione dipendenze mancanti pip3 install rclpy std\_msgs sensor\_msgs # Problema: "Webots Python API not found" # Soluzione: Configurazione PYTHONPATH export PYTHONPATH=$PYTHONPATH:/usr/local/webots/lib/python3.10/site-packages

**⚠️ Troubleshooting Avanzato:**  
Se i problemi persistono, verificare la compatibilità delle versioni software e consultare i log dettagliati in `log/` generati da colcon build.

10\. CONCLUSIONI
================

10.1 Risultati Ottenuti
-----------------------

Il progetto **NAO Library Assistant** ha raggiunto tutti gli obiettivi prefissati, dimostrando un'efficace integrazione tra tecnologie robotiche avanzate:

**🎯 Obiettivi Raggiunti:**

*   **✅ Simulazione Completa**: Sistema robotico completamente funzionale
*   **✅ Integrazione ROS2-Webots**: Bridge stabile e performante
*   **✅ Navigazione Autonoma**: Percorsi ottimizzati e affidabili
*   **✅ Interfaccia Utente**: Interazione intuitiva e responsive

10.2 Impatto Tecnologico
------------------------

Il progetto dimostra come l'integrazione di tecnologie mature (ROS2) con simulatori avanzati (Webots) possa produrre sistemi robotici sofisticati e praticamente utilizzabili. L'approccio modulare adottato rappresenta un modello replicabile per futuri sviluppi nel campo della robotica di servizio.

**🌟 Contributo Scientifico:**  
Questo lavoro contribuisce al corpus di conoscenze sulla **robotica di servizio**, dimostrando metodologie efficaci per l'integrazione di sistemi complessi e fornendo un template riutilizzabile per progetti similari.

10.3 Considerazioni Finali
--------------------------

Il sistema **NAO Library Assistant** rappresenta un esempio concreto di come la robotica moderna possa essere applicata a contesti quotidiani per migliorare l'efficienza e l'esperienza utente. L'integrazione riuscita tra ROS2 e Webots apre la strada a sviluppi futuri sempre più sofisticati nel campo dell'automazione intelligente.

La documentazione completa, la struttura modulare del codice e l'attenzione alla manutenibilità rendono questo progetto non solo un successo tecnico, ma anche un'importante risorsa didattica per lo studio della robotica avanzata.

* * *

**Fine della Relazione Tecnica**  
Sistema NAO Library Assistant con Integrazione ROS2-Webots  
Petrone Kevin, Di Peri Marta, Cirincione Marika  
Giugno 2025
