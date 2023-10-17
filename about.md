---
layout: page
title: whoami
key: page-about
---

<div name="tryhackme"> 
 <script src="https://tryhackme.com/badge/519455"> </script>

  <!-- Right alignment! -->
  <img align="right" width="210" height="200" src="assets/goemon.png" style="border-radius:50%;">
  
</div>
<br><br>

<div name="btlo">
    <h1>Contenuto della Pagina Web</h1>
    <div id="contenuto"></div>

    <script>
        // URL della pagina web che desideri visualizzare
        var url = "[https://www.esempio.com](https://blueteamlabs.online/public/user/z3f1r0)";

        // Seleziona l'elemento div in cui verrà inserito il contenuto
        var contenutoDiv = document.getElementById("contenuto");

        // Effettua una richiesta fetch per ottenere il contenuto della pagina web
        fetch(url)
            .then(response => {
                if (!response.ok) {
                    throw new Error("Errore nella richiesta HTTP: " + response.status);
                }
                return response.text();
            })
            .then(data => {
                // Inserisci il contenuto nella pagina HTML
                contenutoDiv.innerHTML = data;
            })
            .catch(error => {
                console.error("Si è verificato un errore:", error);
            });
    </script>

</div>
