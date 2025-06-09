let pragas = [];
let trigos = [];
let score = 0;
let vidas = 3;
let tempoDeVida = 8000;
let ultimaPraga = 0;
let jogoAtivo = true;
let nivel = 1;
let progressoTrigo = 0; // Variável para o progresso do trigo (crescimento)

function setup() {
  createCanvas(600, 400);
  textAlign(CENTER, CENTER);
  textSize(24);

  // Criar uma fileira de trigos
  for (let i = 60; i < width; i += 60) {
    trigos.push(new Trigo(i, height - 40));
  }
}

function draw() {
  background(100, 200, 100); // cor da lavoura

  if (jogoAtivo) {
    fill(0);
    text("Pontuação: " + score, width / 2, 30);
    text("Vidas: " + vidas, width / 2, 60);
    text("Nível: " + nivel, width / 2, 90);

    // Mostrar barra de progresso do trigo
    fill(255, 204, 0);
    rect(50, height - 30, progressoTrigo * 5, 10);

    // Atualizar e mostrar trigos
    for (let trigo of trigos) {
      trigo.mostrar();
    }

    // Criar novas pragas com base no nível
    if (millis() - ultimaPraga > 1500 - (nivel - 1) * 200) {
      pragas.push(new Praga());
      ultimaPraga = millis();
    }

    // Atualizar pragas
    for (let i = pragas.length - 1; i >= 0; i--) {
      pragas[i].mover();
      pragas[i].mostrar();

      // Verifica colisão com algum trigo
      for (let j = 0; j < trigos.length; j++) {
        if (!trigos[j].comido && pragas[i].colideComTrigo(trigos[j])) {
          trigos[j].comido = true;
          pragas.splice(i, 1);
          vidas--;
          if (vidas <= 0) {
            jogoAtivo = false;
          }
          break;
        }
      }
    }

    // Se todos os trigos forem comidos, o jogo termina
    if (trigos.every(t => t.comido)) {
      progressoTrigo = min(progressoTrigo + 1, 100); // Cresce até 100
      score += 10; // Aumenta a pontuação
      if (progressoTrigo === 100) {
        nivel++;
        // Resetar os trigos e pragas para o próximo nível
        trigos = [];
        for (let i = 60; i < width; i += 60) {
          trigos.push(new Trigo(i, height - 40));
        }
        pragas = [];
      }
    }
  } else {
    fill(0);
    textSize(32);
    text("💀 Fim de Jogo! 💀", width / 2, height / 2 - 20);
    textSize(24);
    text("Pontuação final: " + score, width / 2, height / 2 + 20);
  }
}

// Clique para eliminar pragas e ganhar vidas
function mousePressed() {
  if (!jogoAtivo) return;

  for (let i = pragas.length - 1; i >= 0; i--) {
    if (pragas[i].foiClicada(mouseX, mouseY)) {
      pragas.splice(i, 1);
      score++;

      // Se o jogador tiver menos de 3 vidas, ganha uma vida extra
      if (vidas < 3) {
        vidas++;
      }
      break;
    }
  }
}

// Classe da Praga
class Praga {
  constructor() {
    this.x = random(50, width - 50);
    this.y = 0;
    this.tamanho = 40;
    this.velocidade = random(1, 3);
    this.direcaoX = random([-1, 1]);
    this.direcaoY = random(1, 2);
  }

  mover() {
    // Movimento aleatório da praga
    this.x += this.direcaoX * this.velocidade;
    this.y += this.direcaoY * this.velocidade;
    
    // Impede a praga de sair da tela
    if (this.x > width || this.x < 0) {
      this.direcaoX *= -1;
    }
    if (this.y > height) {
      this.direcaoY *= -1;
    }
  }

  mostrar() {
    fill(80, 0, 0);
    ellipse(this.x, this.y, this.tamanho);
    fill(255);
    text("🐛", this.x, this.y);
  }

  foiClicada(mx, my) {
    let d = dist(mx, my, this.x, this.y);
    return d < this.tamanho / 2;
  }

  colideComTrigo(trigo) {
    return dist(this.x, this.y, trigo.x, trigo.y) < this.tamanho / 2 + trigo.tamanho / 2;
  }
}

// Classe do Trigo
class Trigo {
  constructor(x, y) {
    this.x = x;
    this.y = y;
    this.tamanho = 30;
    this.comido = false;
  }

  mostrar() {
    if (this.comido) return;
    fill(255, 204, 0);
    ellipse(this.x, this.y, this.tamanho, this.tamanho * 1.5);
    fill(0);
    text("🌾", this.x, this.y - 5);
  }
}
