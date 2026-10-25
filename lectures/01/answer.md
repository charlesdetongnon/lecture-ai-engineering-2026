# 第1回 解答

## 演習

- 講座に固有の質問（宿題の3レベル）へのモデルの回答（要約でよい）：
- 用語集の正しい意味と比べて気づいたこと：
- 表示できたトークン数（入力 / 出力）：

## 宿題

取り組んだレベル：基礎

やったこと： J'ai posé 3 fois la même question ( Qu'est-ce que LoRA ?) avec le même modèle (gemini-3.5-flash-lite) en changeant uniquement la température (0, 1, 2) grâce à une option --temperature ajoutée au script app/ask.py. J'ai aussi ajouté dans ask() l'affichage des tokens d'entrée et de sortie à partir de response.usage_metadata. Commande utilisée : uv run python -m app.ask "Qu'est-ce que LoRA ?" --temperature 0 (puis 1, puis 2).

結果：

Température 0 : 10 tokens en entrée, 807 en sortie. 4 sections + conclusion, analogie du dictionnaire et du carnet de notes. Imprécisions : « poids synaptiques », « groupes de neurones », « On ne entraîne ».
Température 1 : 10 tokens en entrée, 584 en sortie. 4 sections, analogie du Post-it sur un livre. Aucune erreur évidente.
Température 2 : 10 tokens en entrée, 536 en sortie. 3 sections + conclusion, analogie du Post-it. Erreurs : « fin gourmet » pour fine-tuning, « rang faible », chiffres vagues.
Comparaison :
La température 0 donne la réponse la plus longue et la plus complète.
Les températures 1 et 2 sont plus courtes et se ressemblent (même analogie, même plan).
La température 2 est la seule à contenir une erreur de vocabulaire flagrante.
Les tokens d'entrée sont identiques (10) puisque la question est la même ; seule la sortie varie.

気づき・疑問： Le glossaire définit la température comme le réglage du caractère aléatoire du choix du prochain token (bas = stable, haut = varié). Mes essais vont dans ce sens : le vocabulaire est plus fiable à température 0 et une erreur apparaît à température 2. Le fond technique (niveau de détail sur LoRA) est resté le même aux trois températures ; la température a surtout changé la longueur, le style et la qualité du vocabulaire. La longueur diminue quand la température augmente (807 → 584 → 536), mais je n'ai fait qu'un seul essai par température, donc je ne peux pas affirmer que c'est une règle générale. Questions : combien d'essais faut-il pour conclure de façon fiable ? Si je relance deux fois la température 0, obtiendrai-je des réponses presque identiques ? Et à température 2, les erreurs comme « fin gourmet » apparaissent-elles à chaque fois ?