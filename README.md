# Smart Air Node — Protótipo

Arquivos executáveis e documentação do **Produto 2** da Smart Air: dispositivo a bateria que liga o ar-condicionado quando há pessoas na sala e desliga após **10 min** sem presença.

O plano de implementação (arquitetura, custos, cronograma, ROI) está em [`../PLANO_IMPLEMENTACAO.md`](../PLANO_IMPLEMENTACAO.md).

## Estrutura

```
SmartAir-Prototype/
├── index.html                  página inicial
├── demo-3d.html                demonstração 3D dos componentes (Three.js)
├── simulacao.html              simulação da sala de aula (Three.js)
├── caixa-3d/
│   ├── smartair_caixa.scad     modelo paramétrico (OpenSCAD)
│   ├── base.stl                base da caixa (imprimir com o fundo na mesa)
│   ├── tampa.stl               tampa com travas (já vem virada, frente na mesa)
│   ├── suporte.stl             suporte de parede com cunha de 15°
│   └── preview_*.png           imagens de pré-visualização
├── firmware/smartair_node/
│   ├── smartair_node.ino       firmware de referência (ESP32-C3, Arduino core 3.x)
│   └── config.h                pinos e parâmetros da lógica
└── docs/
    ├── DOCUMENTACAO_TECNICA.md hardware, ligações, lógica, energia, firmware, MQTT
    └── CAIXA_3D.md             impressão e montagem da caixa preta
```

## Como abrir as demonstrações

As páginas usam módulos ES e carregam o Three.js 0.160 da CDN jsDelivr, então precisam de **internet** e de um **servidor local** (abrir o arquivo com duplo clique não funciona por causa dos módulos).

Na pasta `SmartAir-Prototype`:

```powershell
python -m http.server 8765
```

Depois abra <http://localhost:8765/>.

Atalhos por URL:

| URL | Abre |
|---|---|
| `demo-3d.html?caixa=3d` | demonstração 3D já com a caixa preta impressa |
| `simulacao.html?cenario=leitor` | cenário (`dia`, `leitor`, `falso`, `saida`, `livre`) |
| `simulacao.html?caixa=3d` | dispositivo com a caixa preta na sala |

## Demonstração 3D (`demo-3d.html`)

- **Clique** em qualquer peça, número ou item da lista → painel com função, part number, especificação, tabela de pinos, consumo, custo e observações.
- **Clique em um fio** ou na legenda → descrição do sinal (VBAT, 3V3, VSENS, PIR→GPIO3, TRIG/ECHO, I²C, IR…) e todos os trechos.
- **Explodir**: separa as camadas (fixação → suporte → base → bateria → PCB → sensores → tampa).
- **Carcaça**: opaca / raio-X / oculta.
- **Caixa**: *Produção (branca)* ou *Impressão 3D (preta)*. Na impressa aparecem as 4 travas, as janelas na base, as fechaduras traseiras, o recorte do USB-C e a chave liga/desliga, e os links dos STLs.
- **Na parede**: mostra o dispositivo fixado.

## Simulação (`simulacao.html`)

Sala 8 × 10 m com 30 carteiras, quadro, porta, janelas, luminárias e um split. O Smart Air Node está na parede do fundo, a 2,35 m, inclinado 15°, apontando para o AC.

- **Cenários**: dia letivo completo, pessoa parada estudando, falsos positivos (porta, visita rápida, faxina fora do horário), fim de aula com luz apagada e modo livre.
- **Velocidade** 1× a 900×; pausar e reiniciar.
- **Controle manual**: colocar aluno, leitor dentro/fora do cone do ultrassom, tirar pessoas, passante na porta, luz.
- **Configuração**: contagem (3–30 min), contagem com luz apagada, liga/desliga cada sensor, verificação pelo AHT20, taxa de acerto do IR, potência do AC, tarifa, hardware (produção/protótipo), rádio, caixa (branca/preta).
- **Resultados**: economia (%, kWh, R$) contra uma referência manual, horas de AC, minutos com pessoas e AC desligado, falsos positivos evitados, comandos IR e reenvios, e autonomia projetada da bateria.
- **Linha do tempo**: pessoas, estado do Node, AC Smart Air × AC manual, disparos do PIR, leituras do ultrassom, lux e temperatura.

Resultados com a semente padrão (10 min, todos os sensores):

| Cenário | Smart Air | Referência | Economia | Pessoas com AC desligado |
|---|---|---|---|---|
| Dia letivo (ref. 07:00–22:30) | 11,7 kWh | 17,1 kWh | **32 %** | 0 min |
| Fim de aula com luz apagada | 0,7 kWh | 2,2 kWh | 69 % | 0 min |
| Falsos positivos | 0,1 kWh | 0,2 kWh | 3 falsos positivos rejeitados | 0 min |
| Pessoa parada, **com** ultrassom | — | — | — | **0 min** |
| Pessoa parada, **sem** ultrassom | — | — | — | **117 min** |

A economia depende do padrão de uso da sala. O Smart Place mediu 61,8 % na UFRN porque as salas tinham mais horas vazias que este dia simulado.

### Simplificações do modelo
- Consumo do AC = potência nominal × tempo ligado (mesmo método do estudo do Smart Place).
- PIR: 95 %/s de chance de detectar quem anda e 60 % para um micro-movimento de quem está sentado; alcance de 12 m e ±50°.
- Ultrassom: detecta a pessoa mais próxima dentro de um cone de 15° até 4,5 m.
- Temperatura: modelo de primeira ordem (troca com o exterior + 0,003 °C/min por pessoa − refrigeração).
- Bateria: custos por evento da tabela em `docs/DOCUMENTACAO_TECNICA.md`, projetados para 24 h + autodescarga de 2 mAh/dia.

## Firmware

Veja `docs/DOCUMENTACAO_TECNICA.md` §6. O código é uma **referência ainda não compilada nem testada em hardware**: a primeira tarefa da Fase 0 é compilar, gravar e medir o consumo.

## Caixa impressa em 3D

Veja `docs/CAIXA_3D.md`. Para regenerar os STLs após mudar parâmetros:

```powershell
& "C:\Program Files\OpenSCAD\openscad.com" -o base.stl -D 'part="base"' smartair_caixa.scad
```
