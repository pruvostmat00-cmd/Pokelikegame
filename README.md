<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Mini RPG 2D</title>
  <style>
    * { box-sizing: border-box; user-select: none; -webkit-user-select: none; }
    body {
      margin: 0; background: #1a1a2e; color: white; font-family: sans-serif;
      display: flex; flex-direction: column; align-items: center; justify-content: space-between;
      height: 100vh; padding: 10px;
    }
    #ecran {
      position: relative; width: 320px; height: 320px;
      border: 4px solid #fff; border-radius: 8px; overflow: hidden; background: #7ec850;
    }
    canvas { display: block; width: 100%; height: 100%; }
    
    /* Interface de combat */
    #combat {
      display: none; position: absolute; inset: 0; background: #f8f9fa; color: #222;
      flex-direction: column; justify-content: space-between; padding: 15px;
    }
    .barre-pv { width: 100%; height: 12px; background: #ddd; border-radius: 6px; overflow: hidden; margin-top: 4px; }
    .pv-fill { height: 100%; background: #28a745; transition: width 0.3s; }
    #log-combat { background: #222; color: #fff; padding: 10px; border-radius: 6px; font-size: 14px; min-height: 55px; }
    .actions-combat { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }
    
    /* Manette tactile */
    #controles {
      display: grid; grid-template-columns: repeat(3, 65px); grid-template-rows: repeat(3, 65px);
      gap: 5px; margin-bottom: 15px;
    }
    button {
      background: #30475e; color: white; border: none; border-radius: 12px;
      font-size: 20px; font-weight: bold;
    }
    button:active { background: #f05454; transform: scale(0.95); }
    .btn-combat { font-size: 14px; padding: 12px; background: #007bff; }
    .btn-fuite { background: #6c757d; }
  </style>
</head>
<body>

  <div id="ecran">
    <canvas id="jeu" width="320" height="320"></canvas>
    
    <div id="combat">
      <div>
        <strong id="nom-ennemi">Herbizarre Sauvage</strong> (Niv. 3)
        <div class="barre-pv"><div id="pv-ennemi" class="pv-fill" style="width: 100%;"></div></div>
      </div>
      
      <div style="text-align: right;">
        <strong>Flamèche</strong> (Niv. 5)
        <div class="barre-pv"><div id="pv-joueur" class="pv-fill" style="width: 100%;"></div></div>
      </div>

      <div id="log-combat">Un monstre sauvage apparaît !</div>

      <div class="actions-combat">
        <button class="btn-combat" onclick="attaquer()">⚔️ Attaque</button>
        <button class="btn-combat btn-fuite" onclick="fuir()">🏃 Fuir</button>
      </div>
    </div>
  </div>

  <div id="controles">
    <div></div>
    <button ontouchstart="bouger(0, -1)" onclick="bouger(0, -1)">▲</button>
    <div></div>
    <button ontouchstart="bouger(-1, 0)" onclick="bouger(-1, 0)">◀</button>
    <div></div>
    <button ontouchstart="bouger(1, 0)" onclick="bouger(1, 0)">▶</button>
    <div></div>
    <button ontouchstart="bouger(0, 1)" onclick="bouger(0, 1)">▼</button>
    <div></div>
  </div>

<script>
  const canvas = document.getElementById('jeu');
  const ctx = canvas.getContext('2d');
  const TAILLE = 32;

  // Construction de la carte (0 = herbe, 1 = mur/arbre, 2 = hautes herbes)
  const carte = [];
  for (let y = 0; y < 10; y++) {
    let ligne = [];
    for (let x = 0; x < 10; x++) {
      if (x === 0 || x === 9 || y === 0 || y === 9) {
        ligne.push(1); // Bordure d'arbres autour de la carte
      } else if (x >= 5 && y >= 5) {
        ligne.push(2); // Zone de hautes herbes en bas à droite
      } else {
        ligne.push(0); // Herbe normale
      }
    }
    carte.push(ligne);
  }

  let joueur = { x: 2, y: 2 };
  let enCombat = false;
  let pvJoueur = 100, pvEnnemi = 100;

  function dessiner() {
    for (let y = 0; y < 10; y++) {
      for (let x = 0; x < 10; x++) {
        if (carte[y][x] === 1) ctx.fillStyle = '#2d5a27';
        else if (carte[y][x] === 2) ctx.fillStyle = '#3b8c32';
        else ctx.fillStyle = '#88d463';
        ctx.fillRect(x * TAILLE, y * TAILLE, TAILLE, TAILLE);
        ctx.strokeStyle = 'rgba(0,0,0,0.08)';
        ctx.strokeRect(x * TAILLE, y * TAILLE, TAILLE, TAILLE);
      }
    }
    ctx.fillStyle = '#e63946';
    ctx.fillRect(joueur.x * TAILLE + 4, joueur.y * TAILLE + 4, TAILLE - 8, TAILLE - 8);
  }

  function bouger(dx, dy) {
    if (enCombat) return;
    let nx = joueur.x + dx;
    let ny = joueur.y + dy;
    if (carte[ny] && carte[ny][nx] !== 1) {
      joueur.x = nx;
      joueur.y = ny;
      dessiner();
      if (carte[ny][nx] === 2 && Math.random() < 0.35) {
        lancerCombat();
      }
    }
  }

  function lancerCombat() {
    enCombat = true;
    pvEnnemi = 100;
    document.getElementById('pv-ennemi').style.width = '100%';
    document.getElementById('log-combat').innerText = "Un monstre sauvage apparaît !";
    document.getElementById('combat').style.display = 'flex';
  }

  function attaquer() {
    let degats = Math.floor(Math.random() * 25) + 15;
    pvEnnemi = Math.max(0, pvEnnemi - degats);
    document.getElementById('pv-ennemi').style.width = pvEnnemi + '%';
    
    if (pvEnnemi === 0) {
      document.getElementById('log-combat').innerText = "Victoire ! Le monstre est K.O.";
      setTimeout(() => { document.getElementById('combat').style.display = 'none'; enCombat = false; }, 1200);
      return;
    }

    let riposte = Math.floor(Math.random() * 15) + 5;
    pvJoueur = Math.max(0, pvJoueur - riposte);
    document.getElementById('pv-joueur').style.width = pvJoueur + '%';
    document.getElementById('log-combat').innerText = `Tu infliges ${degats} dégâts ! Il riposte (-${riposte} PV).`;

    if (pvJoueur === 0) {
      document.getElementById('log-combat').innerText = "Tu es K.O. ! Retour au départ...";
      setTimeout(() => {
        pvJoueur = 100;
        document.getElementById('pv-joueur').style.width = '100%';
        joueur = { x: 2, y: 2 };
        dessiner();
        document.getElementById('combat').style.display = 'none';
        enCombat = false;
      }, 1500);
    }
  }

  function fuir() {
    document.getElementById('combat').style.display = 'none';
    enCombat = false;
  }

  dessiner();
</script>
</body>
</html>
