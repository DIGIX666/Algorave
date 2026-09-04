```haskell
// SUNSET PRESSURE — 140 BPM / 64 mesures
// intro 8 | couplet 16 | refrain 16 | couplet 16 | outro 8
// SLIDERS: drive kick | filtre basse | enveloppe basse | FM pad | FM pulse
setcps(140/60/4)

const kick = s("sbd!4")
  .distort(slider(3,0,8,0.1)).postgain(0.30)
  .gain(0.82)
  .duckorbit("2:3:4").duckattack(0.2).duckdepth(0.8)
  .color("orange")._scope({color:"orange",scale:0.5})

const bassRandom = n(irand(10).sub(7).seg(16))
  .scale("C2:minor").s("sawtooth").distort("2.2:.3")
  .lpf(slider(220,80,1200,10))
  .lpenv(slider(2.28,0,8,0.01)).lpq(12)
  .decay(0.12).sustain(0).orbit(2)
  .color("cyan")._pianoroll({cycles:4,labels:1})

const bassHook = n("0 ~ 0 3 5 ~ 7 3 0 ~ 0 3 8 7 5 3")
  .scale("C2:minor").s("sawtooth").distort("2.6:.28")
  .lpf(360).lpenv(3.2).lpq(14)
  .decay(0.11).sustain(0).orbit(2)
  .color("magenta")._pianoroll({cycles:4,labels:1})

const supersawAir = note("<c3 eb3 ab3 g3>")
  .s("supersaw").detune(1).release(5)
  .fm(slider(2,0,8,0.1)).fmh(2.04)
  .room(1).size(4).orbit(3)

const pulseGrid = note("<c5 eb5 g5 bb5>*4")
  .s("pulse").seg(16).decay(0.08).sustain(0)
  .fm(time.mul(slider(1,0,4,0.05))).fmh(time)
  .lpf(1800).gain(0.10).orbit(4)
  .color("yellow")._punchcard({cycles:2})

// INTRO — brume supersaw, grille FM, kick filtré après 4 mesures
const intro = stack(
  supersawAir.slow(2).lpf(sine.range(300,1800).slow(8)).gain(0.13),
  pulseGrid.slow(2).lpf(sine.range(500,2400).slow(8)).mask("<0!2 1!6>"),
  kick.lpf(sine.range(100,900).slow(4)).gain(0.62).mask("<0!4 1!4>")
)

// COUPLET 1 — basse générative et groove sec
const verse1 = stack(
  kick.gain(0.76),
  s("~ cp ~ cp").bank("tr909").gain(0.28),
  s("[~ hh]*4").bank("tr909").hpf(7200).gain(0.13),
  bassRandom,
  pulseGrid.gain(0.07).mask("<1!3 0>")
)

// REFRAIN — basse-hook, mur supersaw, hats ouverts, FM plus présente
const chorus = stack(
  kick.gain(0.92).shape(0.12),
  s("~ [cp cp] ~ cp").bank("tr909").gain(0.44).room(0.15),
  s("hh*8").bank("tr909").hpf(7600).gain("0.10 0.18"),
  s("~ oh ~ oh").bank("tr909").gain(0.24).room(0.24),
  s("cr").bank("tr909").gain(0.26).room(0.5).mask("<1 0!7>"),
  bassHook,
  note("<[c4,eb4,g4] [bb3,d4,f4] [ab3,c4,eb4] [g3,bb3,d4]>")
    .s("supersaw").detune(1.2).attack(0.01).release(1.2)
    .lpf(slider(2600,400,6000,20)).gain(0.16).room(0.55).size(3).orbit(3),
  pulseGrid.gain(0.15).lpf(3200)
).compressor("-18:5:8:.003:.08")._spectrum({speed:2,min:-80,max:0})

// COUPLET 2 — plus nerveux, avec respirations toutes les 8 mesures
const verse2 = stack(
  kick.gain(0.80).mask("<1!7 0>"),
  s("~ cp ~ cp").bank("tr909").gain(0.32).room(0.12),
  s("[~ hh]*4").bank("tr909").hpf(7000).gain(0.15),
  bassRandom.lpf(sine.range(180,520).slow(8)),
  pulseGrid.gain(0.09).every(4, x => x.rev()),
  supersawAir.slow(4).lpf(900).gain(0.07)
)

// OUTRO — on retire la rythmique, puis le filtre se referme
const outro = stack(
  kick.gain(0.66).lpf(sine.range(900,120).slow(8)).mask("<1!4 0!4>"),
  bassHook.lpf(sine.range(320,90).slow(8)).gain(0.22).mask("<1!4 0!4>"),
  supersawAir.slow(2).lpf(sine.range(1600,180).slow(8)).gain(0.11),
  pulseGrid.slow(2).lpf(sine.range(2200,200).slow(8)).gain(0.06).mask("<1!6 0!2>")
)

$: arrange(
  [8, intro],
  [16, verse1],
  [16, chorus],
  [16, verse2],
  [8, outro]
)
```