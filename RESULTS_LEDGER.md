## A. checks / checks-judge — балл перемерен с файла, который лежит на диске сейчас

| # | задача | файл решения | размер | sha256 | мой драйвер | официальный `tests/test.sh` |
|---|---|---|---|---|---|---|
| 1 | `cooling-water-flush-plan` | `/work/cw/ws/output/submission.zip` | 10349 b | `ae3b5b76f6b3e500` | 0.949233 | **0.949233** |
| 2 | `data-center-site-model` | `data-center-site-model/ws/submission/model.ifc` | 10328 b | `8b406beeecdaceee` | 1.000000 | **1.000000** |
| 3 | `data-hall-device-takeoff` | `data-hall-device-takeoff/output/submission.zip` | 8748 b | `626f9303198adf49` | 0.969898 | **0.969898** |
| 4 | `data-hall-mep-routing` | `data-hall-mep-routing/output/solution.py` | 33315 b | `1e1ab19a4b470608` | 1.000000 | **1.000000** |
| 5 | `data-hall-mep-routing-offset` | `data-hall-mep-routing-offset/output/solution.py` | 37507 b | `1bdb60772d38119f` | 1.000000 | **1.000000** |
| 6 | `division-26-specifications` | `division-26-specifications/build/stage2/submission.zip` | 155231 b | `52e1b7f806ab3c52` | 1.000000 | **1.000000** |
| 7 | `eor-review-corrections` * | `eor-review-corrections/build/submission.zip` | 19462 b | `e616192c0212e0b8` | 0.879869 | **0.919869** |
| 8 | `foundation-strategy` * | `foundation-strategy/out/submission.zip` | 33276 b | `6debcd3f2f5b2bd9` | 0.882547 | — (без ключа судьи путь падает до грейдера) |
| 9 | `frame-extension-method` | `frame-extension-method/out/pkg/submission.zip` | 14375 b | `9852b4598dc2fc5d` | 0.806800 | **0.806800** |
| 10 | `heat-rejection-life-cycle-cost` | `heat-rejection-life-cycle-cost/out/submission.zip` | 100582 b | `2f5581ec1171c92d` | 1.000000 | **1.000000** |
| 11 | `lv-cable-sizing` * | `/work/lv-cable-sizing/build3/submission.zip` | 35905 b | `02e84d7896bd87d9` | 0.892650 | — (без ключа судьи путь падает до грейдера) |
| 12 | `mv-conveyance-study` | `mv-conveyance-study/build/submission.zip` | 64509 b | `1041555ac7a62b07` | 0.701837 | **0.701837** |
| 13 | `roof-deck-design` | `roof-deck-design/submission.zip` | 28565 b | `cda37f8626fa1478` | 0.839354 | **0.839354** |
| 14 | `seismic-diaphragm-shear` | `seismic-diaphragm-shear/submission.zip` | 38498 b | `b686556a2137f41b` | 0.904833 | **0.904833** |
| 15 | `wind-diaphragm-shear` | `wind-diaphragm-shear/out/submission.zip` | 54244 b | `34061c2f195f904c` | 0.762579 | **0.762579** |

строк: 15 из 15; официальным путём подтверждено 13; `*` = checks-judge.

**Две не подтверждены официальным путём по одной причине:** в `verify.py` этих задач `LLM_JUDGE = True`, и без `JUDGE_BASE_URL` он выходит **до** запуска грейдера (`GRADER FAILED: JUDGE_BASE_URL is not set`). Их грейдерная часть перемерена драйвером напрямую (`run_grade.py --no-judge` + `paper_score.py`) — те же файлы, тот же код, минус relay судьи.

## B. rubric — пакет собран и лежит на диске; балла нет, судьи нет

| # | задача | файл решения | размер | sha256 | совпал с записью |
|---|---|---|---|---|---|
| 1 | `backfeed-energization-reply` | `backfeed-energization-reply/out/Halbray_Backfeed_Reply.docx` | 43451 b | `f5a90d94121dca7f` | да |
| 2 | `emergency-spillway-check` | `emergency-spillway-check/out/Memo_to_Callaway_Spillway_2026-10-31.docx` | 46094 b | `4280fed48b650054` | да |
| 3 | `engineers-estimate-check` | `engineers-estimate-check/out/Correspondence/Client/Memo_to_Callaway_2026-10-31.docx` | 40771 b | `fdf51ee0701297ff` | да |
| 4 | `fill-compaction-acceptance` | `fill-compaction-acceptance/output/Thelbridge_Materials_Acceptance_Fill_Compaction_2026-0708.docx` | 42525 b | `5e6234c16d9b04ba` | да |
| 5 | `fire-solve-convergence` | `fire-solve-convergence/out/CFDME-2025-041_transient_solve_disposition_revA.docx` | 50471 b | `c1d6cdf28d5c274e` | да |
| 6 | `fire-test-validation-memo` | `fire-test-validation-memo/output/CFDME-2025-041_Validation_Memo_Section7.docx` | 46265 b | `69daa3950ad8c928` | да |
| 7 | `footing-settlement-verification` | `/work/foot/out/Thelbridge_Verification_Memo_Item051_BlockC.docx` | 17638 b | `54fd2aa41d8471f9` | да |
| 8 | `landscape-sheet-revision` | `landscape-sheet-revision/output/DSR_Ph3_L4_Street_Tree_Memo_2026-10-31.docx` | 41169 b | `1c7bed319668fa67` | да |
| 9 | `les-mesh-resolution` | `les-mesh-resolution/out/CFDME-2025-041_resolution_note_revA.docx` | 43314 b | `a0155983ffa83ae4` | да |
| 10 | `permission-to-operate-memo` | `permission-to-operate-memo/out/Corwyn_PTO_Memo.docx` | 42770 b | `42b209e06c73411c` | да |
| 11 | `pre-application-capacity-response` | `pre-application-capacity-response/out/ALN-108_PreApplication_Response.docx` | 46230 b | `ed8039a084c3e826` | да |
| 12 | `shoring-design-verification` | `shoring-design-verification/out/Thelbridge_Verification_Memo_Item048_BlockB.docx` | 17163 b | `a9c7569652780a51` | да |
| 13 | `solar-supplemental-review` | `/work/ssr/build/Bramwick_Commons_Screen_Memo.docx` | 32204 b | `198adf5577db161d` | да |
| 14 | `solar-witness-test-package` | `/work/wit/out/Municipal_Witness_Test_Package.docx` | 45225 b | `7720499c855aca07` | да |

строк: 14 из 14.
