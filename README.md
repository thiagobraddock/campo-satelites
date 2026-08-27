# Desafio: Campo de Satélites

![Campo de Satélites](screenshot.png)

Você já tem um foguete que voa. Agora vamos encher o céu de satélites: se o foguete encostar em um, explode.

Este desafio continua o projeto do foguete que você fez em aula. A lógica dos satélites é a mesma dos canos do Flappy, só que girada: no Flappy os obstáculos vinham da direita para a esquerda, aqui eles caem de cima para baixo.

Faça um passo de cada vez. Cada passo termina com um TESTE: só avance quando ele funcionar.

---

## Passo 1. Abrir o projeto e conferir

Este repositório já vem com o foguete funcionando. Baixe a pasta (ou clone o repositório), abra no VS Code, abra o terminal (menu Terminal > New Terminal) e rode:

```bash
npm install
npm run dev
```

O `npm install` baixa o Phaser e o Vite; roda só uma vez. O `npm run dev` liga o servidor. Abra o endereço que aparecer no terminal (normalmente http://localhost:5173).

**TESTE, confira as quatro coisas:**

1. O foguete aparece na tela e cai até o chão
2. Segurar espaço faz ele subir; soltar, cair
3. A chama acende quando o motor liga e apaga quando desliga
4. As setas esquerda e direita movem o foguete para os lados

Se alguma falhar, abra o console do navegador com **F12** e leia a mensagem de erro: ela diz o arquivo e a linha do problema.

### Onde fica cada coisa

```
campo-de-satelites/
├── index.html              a página que hospeda o jogo
├── style.css               centraliza o jogo na tela
├── main.js                 ← É AQUI QUE VOCÊ VAI TRABALHAR
├── package.json            lista as dependências
└── public/assets/
    └── foguete.png         a imagem do foguete (3 quadros)
```

Abra o `main.js` e leia com calma antes de mexer. Ele tem três partes:

- `preload()` — carrega a imagem do foguete
- `create()` — monta o foguete, o motor e as setas
- `update()` — roda 60 vezes por segundo aplicando o empuxo e o movimento lateral

Todo o código que você vai escrever entra nesse arquivo.

---

## Passo 2. Desenhar o satélite

O satélite não precisa de imagem: vamos desenhar por código, do mesmo jeito que o cano do Flappy foi feito.

Funciona assim: `Graphics` é um pincel. Você escolhe uma cor com `fillStyle`, desenha formas, e no final "fotografa" o desenho com `generateTexture`, dando um nome a ele. Depois o pincel é descartado.

Cole isto no final do `create()`:

```js
// --- textura do satélite (desenhada por código) ---
const g = this.add.graphics();

g.fillStyle(0x9aa4b8);        // cinza: os painéis solares
g.fillRect(0, 8, 14, 8);      // painel da esquerda
g.fillRect(36, 8, 14, 8);     // painel da direita

g.fillStyle(0x378add);        // azul: o brilho dos painéis
g.fillRect(2, 10, 10, 4);
g.fillRect(38, 10, 10, 4);

g.fillStyle(0xd8d8e8);        // branco: o corpo
g.fillRect(15, 2, 20, 20);

g.fillStyle(0xe24b4a);        // vermelho: a anteninha
g.fillRect(23, 0, 4, 4);

g.generateTexture('satelite', 50, 24);   // vira a textura chamada 'satelite'
g.destroy();                             // o pincel já cumpriu o papel
```

Entendendo os números: `fillRect(x, y, largura, altura)` desenha um retângulo. As coordenadas são dentro do desenho, não da tela: o desenho inteiro tem 50 de largura por 24 de altura, como você declara no `generateTexture`. As cores usam `0x` seguido do mesmo código hexadecimal do CSS.

**TESTE:** nada muda na tela. É esperado! A textura só entrou no estoque; ninguém foi colocado em cena ainda.

**Depois de testar, brinque:** mude as cores, aumente o corpo, acrescente outra antena. Não tem certo nem errado aqui.

---

## Passo 3. Criar o grupo e o timer

Um **grupo** é uma caixa que guarda vários objetos parecidos, para você comandar todos de uma vez. Um **timer** executa uma função de tempos em tempos.

Cole no final do `create()`:

```js
// --- os satélites ---
this.satelites = this.physics.add.group();

// a cada 1200 milissegundos, cria um satélite novo
this.time.addEvent({
  delay: 1200,
  loop: true,
  callback: () => this.criarSatelite(),
});
```

**TESTE:** o console vai acusar que `criarSatelite` não existe. Isso é esperado: pedimos ao timer para chamar uma função que ainda não escrevemos. Leia o erro antes de continuar, é assim que se depura.

---

## Passo 4. Fazer os satélites caírem

Agora criamos a função que o timer chama. Ela é um **método da classe**, irmão do `update()` — repare que ela fica DENTRO da classe, mas FORA do `create()`.

```js
criarSatelite() {
  // sorteia a posição horizontal (entre 40 e 400)
  const x = Phaser.Math.Between(40, 400);

  // o satélite nasce ACIMA da tela (y negativo) e vai descer
  const satelite = this.satelites.create(x, -30, 'satelite');

  satelite.body.setAllowGravity(false);  // não acelera: cai em ritmo constante
  satelite.setVelocityY(150);            // desce a 150 pixels por segundo
}
```

Compare com o `criarParDeCanos()` do Flappy e responda no caderno:

1. No Flappy sorteávamos a posição vertical e fixávamos a horizontal. Aqui é o contrário. Por quê?
2. Por que a velocidade agora é positiva, e no Flappy era negativa?

**TESTE:** satélites descendo em posições sorteadas. O foguete atravessa eles como um fantasma.

---

## Passo 5. Limpar o lixo

Os satélites que saem pela parte de baixo continuam existindo para sempre, ocupando memória. Depois de alguns minutos, o jogo trava.

No Flappy você resolveu isso com uma linha dentro do `update()`. Aqui é a mesma ideia, trocando o eixo. Cole no final do `update()`:

```js
// destrói os satélites que já passaram do chão
this.satelites.getChildren().forEach((satelite) => {
  if (satelite.y > 640) satelite.destroy();
});
```

Por que 640, e não 600? Porque a tela termina em 600, e queremos ter certeza de que o satélite saiu inteiro antes de sumir com ele.

**TESTE:** nada muda visualmente. Mas agora o jogo aguenta rodar por muito tempo.

---

## Passo 6. A explosão

Agora a parte que você já sabe fazer. Duas coisas:

**Primeiro**, declare a colisão. É uma linha só, no final do `create()`, ligando o foguete ao grupo de satélites. Você escreveu exatamente essa linha no Flappy (lá era o pássaro e os canos) — procure lá e adapte, chamando um método `explodir()`.

**Segundo**, crie o método `explodir()` na classe. Ele precisa:

1. Ter, na primeira linha, a trava para não explodir duas vezes: `if (this.acabou) return;`
2. Marcar `this.acabou = true`
3. Pausar a física
4. Pintar o foguete de vermelho
5. Mostrar um texto de fim de jogo no meio da tela

Tudo isso você fez no `fimDeJogo()` do Flappy. Consulte e adapte.

**Não esqueça dos dois guardas:**

- `if (this.acabou) return;` na primeira linha do `update()`
- No método que liga o motor: se o jogo acabou, chame `this.scene.restart()` e saia com `return`

**TESTE FINAL:** o foguete sobe, desvia dos satélites com as setas, e explode se encostar. O clique recomeça o jogo.

---

## Passo 7. Mostrar o placar na tela

Um jogo sem placar não tem graça: ninguém sabe se jogou bem. Nosso placar vai contar **quantos segundos você sobreviveu**.

São duas partes: primeiro criar o texto, depois fazer ele contar.

### 7.1 Criar o texto

Cole no final do `create()`:

```js
// --- placar ---
this.segundos = 0;   // o número que vamos contar

// o texto que mostra esse número na tela
this.textoPlacar = this.add
  .text(220, 40, '0s', {
    fontSize: '36px',
    color: '#ffffff',
    fontFamily: 'Arial',
  })
  .setOrigin(0.5)   // o ponto (220, 40) é o CENTRO do texto
  .setDepth(10);    // desenha por cima dos satélites
```

Repare que são **duas coisas diferentes**:

- `this.segundos` é o **número** (o dado)
- `this.textoPlacar` é o **objeto de texto** na tela (o desenho)

Mudar o número não muda o desenho sozinho. Nós que temos que mandar o desenho se atualizar. É isso que a próxima parte faz.

O `setDepth(10)` existe porque quem é desenhado por último fica por cima. Como os satélites nascem depois do placar, sem o depth eles passariam na frente do número.

**TESTE:** aparece um `0s` no alto da tela. Ele ainda não conta.

### 7.2 Fazer o tempo passar

Precisamos que alguém conte de segundo em segundo. Você já conhece a ferramenta certa: o **timer**, o mesmo que usamos para criar os satélites. A diferença é o `delay`.

Cole também no final do `create()`:

```js
// a cada 1000 milissegundos (1 segundo), soma 1 no placar
this.time.addEvent({
  delay: 1000,
  loop: true,
  callback: () => {
    if (this.acabou) return;          // parou o jogo? para de contar

    this.segundos = this.segundos + 1;          // muda o NÚMERO
    this.textoPlacar.setText(this.segundos + 's');  // atualiza o DESENHO
  },
});
```

O `setText()` é o método que troca o conteúdo de um texto já existente na tela. O `+ 's'` gruda a letra s no número, para aparecer `7s` em vez de `7`.

**TESTE:** o número sobe de 1 em 1 enquanto você joga, e congela quando o foguete explode.

**Se o número continuar subindo depois da explosão:** faltou o `if (this.acabou) return;` dentro do callback. O timer não sabe que o jogo acabou; você precisa avisar.

### 7.3 Mostrar o resultado no fim de jogo

No seu método `explodir()`, o texto de fim de jogo já existe. Falta acrescentar o tempo final. Troque a mensagem por algo assim:

```js
this.add.text(220, 300, 'BOOM!\nvocê sobreviveu ' + this.segundos + ' segundos\nclique para tentar de novo', {
  fontSize: '24px',
  color: '#ff4444',
  fontFamily: 'Arial',
  align: 'center',
}).setOrigin(0.5).setDepth(10);
```

O `\n` quebra a linha, e o `+` junta pedaços de texto com o valor da variável.

**TESTE FINAL:** o placar conta durante o jogo, congela na explosão, e a tela de fim de jogo mostra quantos segundos você aguentou.

---

## Está pronto quando

- [ ] O satélite foi desenhado por código e está diferente do exemplo
- [ ] Satélites caem em posições sorteadas
- [ ] Os satélites que saem da tela são destruídos
- [ ] Encostar em um satélite explode o foguete e mostra o fim de jogo
- [ ] O placar conta os segundos e para quando o jogo acaba
- [ ] A tela de fim de jogo mostra o tempo final
- [ ] Clicar reinicia o jogo (e o placar volta a zero)

---

## Desafios extras

Terminou? Escolha um:

**1. Satélites girando.** Adicione uma linha no `criarSatelite()`:

```js
satelite.setAngularVelocity(60);
```

Teste com outros valores. O que acontece com número negativo?

**2. Velocidade variada.** Em vez de todos caírem a 150, sorteie a velocidade de cada um entre 100 e 250. Dica: você já sabe sortear números.

**3. Chuva mais forte.** Diminua o `delay` do timer e veja o jogo ficar mais difícil. Qual valor deixa o jogo impossível?

**4. Recorde.** Guarde o melhor tempo e mostre na tela de fim de jogo. O problema: ao reiniciar, o `create()` roda de novo e apaga tudo. A solução é o `registry`, um armário global que sobrevive ao restart:

```js
this.registry.set('recorde', valor);   // guarda
this.registry.get('recorde');          // lê (devolve undefined na primeira vez)
```

**5. Dificuldade crescente.** A cada 10 segundos, faça os satélites caírem mais rápido. Dica: você já tem um contador de segundos rodando.

**6. Seu próprio foguete.** Desenhe um foguete novo e substitua o `public/assets/foguete.png` (mantenha 3 quadros de 64 por 96, imagem total de 192 por 96).

---

## Referências

- [Documentação do Phaser](https://docs.phaser.io)
- [Exemplos de código](https://phaser.io/examples)
