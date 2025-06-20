document.addEventListener("DOMContentLoaded", () => {
  const biensContainer = document.getElementById("biens-container");
  const filtreForm = document.getElementById("filtre-form");

  const compteurResultats = document.createElement("p");
  compteurResultats.id = "compteur-resultats";
  filtreForm.insertAdjacentElement("afterend", compteurResultats);

  const boutonReset = document.createElement("button");
  boutonReset.type = "button";
  boutonReset.textContent = "Réinitialiser";
  boutonReset.className = "reset-button";
  filtreForm.appendChild(boutonReset);

  let allBiens = [];

  fetch("biens.json")
    .then((res) => res.json())
    .then((biens) => {
      allBiens = biens;
      afficherBiens(biens);
    });

  filtreForm.addEventListener("submit", (e) => {
    e.preventDefault();
    filtrerBiens();
  });

  boutonReset.addEventListener("click", () => {
    filtreForm.reset();
    afficherBiens(allBiens);
    compteurResultats.textContent = `Total : ${allBiens.length} biens trouvés`;
    history.replaceState(null, '', `#resultats=${allBiens.length}`);
  });

  function filtrerBiens() {
    const formData = new FormData(filtreForm);
    const filtre = Object.fromEntries(formData.entries());
    const filtres = {
      ville: filtre.ville.toLowerCase(),
      type: filtre.type,
      chambres: parseInt(filtre.chambres) || 0,
      surface: parseInt(filtre.surface) || 0,
      terrain: parseInt(filtre.terrain) || 0,
      prixMin: parseInt(filtre["prix-min"]) || 0,
      prixMax: parseInt(filtre["prix-max"]) || Infinity
    };
    const resultats = allBiens.filter((bien) => {
      return (
        (!filtres.ville || bien.ville.toLowerCase().includes(filtres.ville)) &&
        (!filtres.type || bien.type === filtres.type) &&
        bien.chambres >= filtres.chambres &&
        bien.surface >= filtres.surface &&
        bien.terrain >= filtres.terrain &&
        bien.prix >= filtres.prixMin &&
        bien.prix <= filtres.prixMax
      );
    });
    afficherBiens(resultats);
    compteurResultats.textContent = `${resultats.length} bien(s) trouvé(s)`;
    history.replaceState(null, '', `#resultats=${resultats.length}`);
  }

  function afficherBiens(biens) {
    biensContainer.innerHTML = "";
    if (biens.length === 0) {
      biensContainer.innerHTML = "<p>Aucun bien ne correspond à votre recherche.</p>";
      return;
    }
    biens.forEach((bien) => {
      const div = document.createElement("div");
      div.className = "bien-card";
      div.innerHTML = `
        <img src="${bien.image}" alt="${bien.titre}">
        <h3>${bien.titre}</h3>
        <p>${bien.ville} - ${bien.surface} m² - ${bien.chambres} ch. - ${bien.terrain} m²</p>
        <p class="prix">${bien.prix.toLocaleString()} €</p>
        <a href="detail.html?id=${bien.id}">Voir le bien</a>
      `;
      biensContainer.appendChild(div);
    });
  }
});
