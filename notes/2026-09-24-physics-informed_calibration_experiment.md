# Results
![[WhatsApp Image 2026-09-24 at 13.32.53.jpeg]]


- Extracted results from the table above:


| Terrain Gap (mm) | Distance cycled (m) | Intended speed | Measured time to complete (sample 1) - (s) | Measured time to complete (sample 2) - (s) | Measured time to complete (sample 3) - (s) |
| ---------------- | ------------------- | -------------- | ------------------------------------------ | ------------------------------------------ | ------------------------------------------ |
| 100              | 10                  | Slow           | 3.63                                       | 4.06                                       | 4.03                                       |
| 100              | 10                  | Medium         | 2.50                                       | 2.45                                       | 2.60                                       |
| 100              | 10                  | Fast           | 1.95                                       | 1.85                                       | 2.06                                       |

Terrain gaps measured (to estimate frequencies based on the data collected above):

- 100mm (the terrain where we did the experiment above)
- 300mm
- 500mm (this would constitute the lowest frequency - worst case scenario for a possible overlap with the other low frequency components of the signal)



Leg Frequency (RPM) estimation

videos saved at `/photos-videos`

| Video           | duration | Leg motion <br>cycles counted |
| --------------- | -------- | ----------------------------- |
| rpm-video-1.mp4 | 17 sec   | 26                            |
| rpm-video-2.mp4 | 21 sec   | 30                            |
| rpm-video-3.mp4 | 25 sec   | 30                            |



# Planning



Sim. Esse experimento faz bastante sentido e, mais importante, dá a vocês uma **justificativa física e empírica** para as bandas de frequência, em vez de escolher um cutoff porque “o gráfico parecia bom”.

A forma como eu enquadraria isso é: vocês estão fazendo um pequeno **physics-informed calibration experiment** antes do ML.

A relação principal é muito simples. Se as irregularidades estão espaçadas por λ\lambda metros e a bicicleta passa a velocidade vv, a frequência aproximada com que elas excitam a bicicleta é:
$$
f_{\text{road}} \approx \frac{v}{\lambda}
$$
Então, no teu exemplo de blocos a cada **0,30 m**, marcando um percurso de 10 m:

There are approximately:

$$
\frac{10}{0.30} \approx 33
$$

transitions.

If the bicycle travels 10 m in 4 s:

$$
v = \frac{10}{4} = 2.5\ \text{m/s}
$$

Therefore:

$$
f_{\text{road}} = \frac{v}{\lambda}
= \frac{2.5}{0.30}
\approx 8.3\ \text{Hz}
$$

- existem aproximadamente $10/0.30≈3310/0.30 \approx 33$ transições;

- se tu leva 4 s para percorrer os 10 m, $v=2.5v=2.5 m/s e f≈8.3f\approx8.3 Hz$;

- se leva 2.5 s, $v=4v=4 m/s e f≈13.3f\approx13.3 Hz$;

- se leva 2 s, $v=5v=5 m/s e f≈16.7f\approx16.7 Hz$.


Isso já cria uma previsão antes mesmo de olhar o acelerômetro.

E vocês podem fazer exatamente a mesma coisa com a perna. Se contarem ciclos completos de pedal durante um intervalo:
$$
fpedal= \frac{N_{\text{cycles}}}{T}
$$
ou, se pensarem em cadence: $fpedal=\frac{\text{RPM}}{60}$

Uma cadência de 60 rpm dá **1 Hz**, 90 rpm dá **1.5 Hz**, 120 rpm dá **2 Hz**. Então a tua hipótese de que o movimento de pedal tende a ocupar uma região muito mais baixa que a excitação de um piso de blocos é bastante plausível.

O experimento mais interessante, porém, é justamente procurar o **pior caso**, como tu falou: bloco bem espaçado + velocidade baixa versus pedalada rápida. É aí que vocês descobrem empiricamente se existe uma zona de overlap.

Eu montaria algo bem enxuto:

1. Escolham 2–3 superfícies com espaçamentos físicos diferentes, por exemplo ~0.30 m, ~0.50 m e alguma irregularidade menos periódica.

2. Meçam aproximadamente o espaçamento λ\lambda.

3. Marquem 10 ou 20 m.

4. Façam passagens lenta, média e rápida.

5. Um colega cronometra o percurso, permitindo calcular $v=L/T$.

6. Calculem a frequência prevista $v/\lambda$

7. Filmagem lateral ou contagem visual da pedalada fornece a frequência de cadence.

8. Ao mesmo tempo, gravem Accelerometer/Gravity/Gyroscope normalmente.

9. Depois comparem as frequências previstas com os picos/energia observados na PSD.


A tabela experimental poderia ser tão simples quanto:

(these are not real values, were AI generated - the real ones should be calculated based on the experiment values)

|Surface|Spacing|10 m time|Speed|Predicted road freq.|Pedal freq.|
|---|--:|--:|--:|--:|--:|
|Block A|0.30 m|4.0 s|2.50 m/s|8.3 Hz|1.2 Hz|
|Block A|0.30 m|2.5 s|4.00 m/s|13.3 Hz|1.6 Hz|
|Block B|0.50 m|4.0 s|2.50 m/s|5.0 Hz|1.2 Hz|

Daí vocês conseguem escrever uma justificativa muito boa:

> Based on the measured spatial periodicity of representative road surfaces and experimentally measured cycling speeds, the expected road-induced excitation frequency was estimated using $f=v/\lambda$. Pedalling cadence was independently estimated from video observations. These measurements were then compared with the measured sensor spectra to identify frequency regions dominated by rider motion and road-surface excitation.

Isso fica muito mais defensável do que simplesmente:

> “We selected a 3 Hz high-pass filter.”

Tem, porém, algumas nuances importantes.

Primeiro, os tijolos **não vão produzir uma senoide perfeita** numa única frequência. Cada junta é mais parecida com um pequeno impulso. Uma sequência de impulsos periódicos produz uma frequência fundamental mais **harmônicos**. Então vocês devem esperar energia em torno de v/λv/\lambda, mas também em múltiplos dessa frequência e espalhada por uma banda.

Segundo, existe todo o sistema mecânico:

**road → tyre → wheel → bicycle frame → rider → phone pocket**

Cada elemento filtra/amplifica certas frequências. Portanto a frequência geométrica prevista é uma referência, não necessariamente exatamente o maior pico do accelerometer.

Terceiro, existe uma particularidade interessante da bicicleta: **roda dianteira e traseira passam pela mesma junta em momentos diferentes**. Isso pode criar uma estrutura adicional no sinal, dependendo de velocidade e wheelbase. Eu não tentaria modelar isso agora, mas mencionaria como uma razão para esperar que o espectro real seja mais complexo que simplesmente uma linha em $f=v/\lambda$.

E quarto: o movimento da perna também não vai ter somente uma frequência. A cadence terá fundamental + harmônicos e haverá movimento do tronco, bicicleta etc. Por isso o que vocês querem demonstrar não é:

> “Leg motion = exatamente 1.4 Hz e road = exatamente 12.7 Hz.”

Mas algo mais realista:

> **A maior parte da energia relacionada à movimentação corporal está concentrada numa região mais baixa, enquanto a rugosidade da superfície introduz energia significativa numa região superior.**

Depois vocês verificam essa hipótese com os sensores.

Isso inclusive te dá uma maneira muito boa de escolher o cutoff sem “chutar”:

**física prevê faixa → experimento controlado confirma → PSD real confirma → só então define-se a faixa de filtragem.**

E eu não escolheria necessariamente um único high-pass rígido imediatamente. Primeiro faria algo como bandas:

**0–3 Hz | 3–10 Hz | 10–20 Hz | 20–40 Hz**

e calcularia a energia em cada uma. Pode ser que vocês descubram que a feature mais discriminante não é simplesmente “everything above 3 Hz”, mas uma razão do tipo:

$$
\frac{P_{10-20 Hz}}{P_{0-3 Hz}}
$$

ou a soma de energia em determinadas bandas.

Isso seria ainda melhor porque fica fisicamente explicável.

E tem um benefício extra: esse experimento testa exatamente a pergunta que vocês deveriam fazer antes de assumir o filtro:

> **Em quais condições a hipótese de separação em frequência deixa de funcionar?**

Se vocês encontrarem overlap em baixa velocidade + espaçamento grande + cadence alta, ótimo. Isso não destrói a abordagem. Significa apenas que naquela região vocês talvez precisem combinar **frequency features + RMS + jerk + gyroscope**, em vez de depender de um cutoff sozinho.

Eu faria esse experimento. É pequeno, barato, reproduzível e transforma uma intuição de engenharia em uma hipótese mensurável. Para esse assignment, é uma justificativa metodológica muito mais forte do que coletar simplesmente mais horas de dados.