# Receita Kling — o padrão (plano único, guia AVCC integral)

Validada no **poa2, 16:9**. Melhor resultado de todos os testes, e **100% compatível com o
documento AVCC** — não precisa de multi-shot nem de slow motion, que eram o conflito aberto.

Clipe de referência: `~/projetos/output/videoanima/avcc-wave-tank/ab/kling-poa2-16x9-B-maquete.mp4`
Script: `scripts/render_kling.sh` · Prompt: `prompts/avcc-ondas-16x9-en.txt`

```bash
scripts/kf_a.py doc/poa2.jpg out/kf_a.png --ratio 16x9 --efeito onda \
  --lock "the same brick power-plant hall, the same tall chimney, the same skyline..."
scripts/render_kling.sh out/kf_a.png out/filme.mp4 prompts/avcc-ondas-16x9-en.txt
```

## Os dois ingredientes

1. **Frame inicial = a maquete já montada no tanque** (`kf_a.png`), não a foto de referência.
   Gerado antes na Agnes (`agnes-image-2.1-flash`, img2img a partir da foto, `size` 1312x736
   para 16:9 / 736x1312 para 9:16). **Este é o ingrediente decisivo** — ver o teste A abaixo.
2. **Prompt = o guia AVCC §5 integral + os 15 negativos do §6.** Texto completo, incluindo o
   bloco de autenticidade de celular (rolling shutter, focus breathing, exposição) e
   "realistic weight, friction, collision, and water resistance" nos destroços.

Sem `--tailImage` e sem `prefer_multi_shots`: plano único contínuo, como o documento exige.
O ratio do vídeo vem do ratio do `kf_a` — o `kling-video-v3_0` não tem argumento de aspect ratio.

## A prova: teste A vs teste B (mesmo prompt, mesma imagem de origem)

| | A — `poa2.jpg` cru | B — `kf_a.png` (maquete) |
|---|---|---|
| abertura | foto real de POA, intacta | estúdio, maquete fiel (chaminé, skyline, orla) |
| destruição | prédios se **dissolvem no lugar**, em cubos, sem onda encostar | prédios **quebram em pedaços** e são arrastados |
| onda | nenhuma | vem do lado da água aberta, atravessa lateralmente |
| final | tanque vazio (chegou lá sem mostrar maquete nem onda) | base raspada + destroços — o "empty flooded foundation" |

Conclusão medida: **o problema era o frame inicial, não o texto.** Com a imagem crua o Kling
entende "transformar esta foto", não "construir uma maquete disto" — e nem o prompt integral
com os 15 negativos vira esse jogo (o próprio negativo "buildings dissolving instead of
physically breaking" foi violado no teste A).

## O que ainda falha na B

- A onda sai **mais baixa** que o pedido ("significantly taller than the miniature buildings").
- O impacto acontece rápido demais; a **segunda surge não fica legível**.
- Sobram estruturas de pé (aceito pelo usuário: "não tem tanto problema").

Hipótese para a próxima iteração: compor o `kf_a` com a maquete encostada em UMA borda e uma
faixa larga de água aberta num lado só, para dar espaço de corrida à onda.

## Operação

Sempre `--poll 0` na submissão + `kling query_tasks` para buscar o resultado. O poll longo do
CLI cai sozinho (`MCP error -32001: Request timed out` / `MCP connection failed: fetch failed`)
enquanto o job termina normalmente — 3 de 9 jobs caíram assim, e piora com jobs em paralelo.
O job é pago e não cancela: grave a resposta crua **antes** de qualquer parse, porque perder o
`generation_id` é perder o job. URLs de resultado expiram em 24h — baixe na hora.

## Parâmetros do `kling-video-v3_0`

| | |
|---|---|
| `duration` | 3 a 15s |
| `resolution` | `720p` · `1080p` · `4k` |
| `enable_audio` | `true` → áudio sincronizado no mesmo passe (o roteiro tem as falas do diretor) |
| `prefer_multi_shots` | **`false`** aqui — `true` introduz cortes, proibidos pelo guia |
| `--tailImage` | **não usar** — ver a variante em `variante-kling-v2.md` |
| aspect ratio | não existe como argumento: **vem do ratio do `kf_a`** |
