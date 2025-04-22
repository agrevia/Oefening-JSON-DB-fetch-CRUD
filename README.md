# Oefening: JSON DB fetch (CRUD) - schoolvakken

## 🗂️ Projectstructuur

```
├── assets/
│   ├── css/
│   │   └── style.css
│   └── js/
│       └── script.js
├── services/
│   └── db.json
└── index.html
```

De server start je met:

```
json-server --watch services/db.json --port 5688
```

------

## 🔧 Stap 1: HTML opbouw (`index.html`)

```
<!DOCTYPE html>
<html lang="nl">
<head>
    <meta charset="UTF-8">
    <title>Vakkenbeheer</title>
    <link rel="stylesheet" href="assets/css/style.css">
</head>
<body>
    <h1>Mijn vakken</h1>

    <!-- Inputveld waarin je de naam van het vak typt -->
    <input type="text" id="vakInput" placeholder="Voer een vak in...">

    <!-- Knop om het vak toe te voegen -->
    <button id="addButton">Toevoegen</button>

    <!-- Hier komt de lijst met vakken -->
    <ul id="vakList"></ul>

    <!-- JavaScript wordt op het einde ingeladen -->
    <script src="assets/js/script.js"></script>
</body>
</html>
```

🔍 **Wat gebeurt hier?**

- Je maakt een tekstveld om de vaknaam in te geven.
- Je voegt een knop toe om het vak te bewaren.
- Je maakt een lijst (`ul`) aan die we straks dynamisch opvullen met `li`-elementen (één per vak).

------

## 💾 Stap 2: De databank (`services/db.json`)

```
{
  "vakken": [
    {
      "id": 1,
      "naam": "Webontwikkeling"
    }
  ]
}
```

📌 **Opmerking:** JSON Server gebruikt deze `id` automatisch om vakken uniek te maken.

------

## 🔌 Stap 3: JavaScript stap per stap (`script.js`)

```
"use strict";

document.addEventListener("DOMContentLoaded", init);

// 1. Init-functie wordt opgeroepen zodra de pagina geladen is
function init() {
    console.log("De pagina is volledig geladen");

    // Koppelt de knop aan de functie addVak
    document.querySelector("#addButton").addEventListener("click", addVak);

    // Haalt bestaande vakken op uit de databank
    fetchVakken();
}
```

------

### 🔄 `fetchVakken()` – haalt alle vakken op

```
async function fetchVakken() {
    try {
        // Ophalen van de vakken uit de JSON-server
        let response = await fetch("http://localhost:5688/vakken");

        // Omzetten naar JSON zodat we ermee kunnen werken
        let vakken = await response.json();

        // Doorsturen naar een functie die de lijst weergeeft
        displayVakken(vakken);
    } catch (err) {
        console.error("Fout bij het ophalen van vakken:", err);
    }
}
```

📌 **Wat doet deze functie?**

- Contacteert de server via HTTP (`fetch`).
- Zet de response om naar JSON.
- Roept een andere functie op om de vakken op het scherm te tonen.

------

### ➕ `addVak()` – voegt een nieuw vak toe

```
async function addVak() {
    let input = document.querySelector("#vakInput");
    let nieuweNaam = input.value.trim();

    if (nieuweNaam === "") {
        alert("Vul een vaknaam in!");
        return;
    }

    try {
        // Verstuur nieuw vak via POST naar de server
        let response = await fetch("http://localhost:5688/vakken", {
            method: "POST",
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify({ naam: nieuweNaam })
        });

        if (response.ok) {
            // Leeg het inputveld en vernieuw de lijst
            input.value = "";
            fetchVakken();
        }
    } catch (err) {
        console.error("Fout bij toevoegen:", err);
    }
}
```

📌 **Wat gebeurt hier?**

- Neemt de waarde uit het tekstveld.
- Controleert of het veld niet leeg is.
- Stuurt een POST-request naar de server.
- Vernieuwt de lijst na succesvolle toevoeging.

------

### ❌ `deleteVak(id)` – verwijdert een vak

```
async function deleteVak(id) {
    try {
        let response = await fetch(`http://localhost:5688/vakken/${id}`, {
            method: "DELETE"
        });

        if (response.ok) {
            fetchVakken(); // vernieuw de lijst na verwijderen
        }
    } catch (err) {
        console.error("Fout bij verwijderen:", err);
    }
}
```

📌 **Wat doet dit?**

- Stuurt een DELETE-request naar de juiste URL (met het id van het vak).
- Vernieuwt de lijst na succesvolle verwijdering.

------

### 📋 `displayVakken(vakken)` – toont de vakken in de browser

```
function displayVakken(vakken) {
    let lijst = document.querySelector("#vakList");
    lijst.innerHTML = ""; // Maak eerst alles leeg

    // Loop over alle vakken
    vakken.forEach(vak => {
        let li = document.createElement("li");
        li.textContent = vak.naam;

        // ❌ knopje maken
        let deleteBtn = document.createElement("button");
        deleteBtn.textContent = "❌";
        deleteBtn.addEventListener("click", () => deleteVak(vak.id));

        li.appendChild(deleteBtn);
        lijst.appendChild(li);
    });
}
```

📌 **Uitleg:**

- Leegt de huidige lijst.
- Voor elk vak maak je een `li` aan met een tekst en een verwijderknop.
- De knop verwijdert het vak via `deleteVak()`.

------

## ✅ Testen

1. Start de JSON server:

   ```
   json-server --watch services/db.json --port 5688
   ```

2. Open `index.html` in je browser.

3. Voeg een vak toe, bekijk de lijst, verwijder eventueel iets.
