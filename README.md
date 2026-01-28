# HackingLLM

L'objectif de ce projet est de créer un assistant (chatbot) spécialisé en cybersécurité capable de fournir des réponses techniques sans les restrictions de sécurité natives, tout en optimisant le modèle pour une exécution fluide.
Pour cela je présente ici une implémentation de la technique d' "abliterration" appliquée au modèle Llama-3.2-3B-Instruct. 

## Qu'est-ce que l'Abliterration ?
Le projet repose sur la méthode décrite par Maxime Labonne dans l'article [*Abliterating safety training from LLMs*](https://huggingface.co/blog/mlabonne/abliteration) Contrairement au fine-tuning classique qui nécessite un réentraînement coûteux, l'abliterration est une technique de "chirurgie" des poids du modèle.

## Méthodologie

### 1. Identification du "Vecteur de Refus"
Le principe est d'identifier dans l'espace latent du modèle une direction spécifique qui correspond au comportement de refus.
* On compare les activations du modèle face à deux jeux de données : des requêtes **harmful** (néfastes) et **harmless** (inoffensives).
* La différence de moyenne entre ces activations permet d'isoler un vecteur mathématique : la **direction du refus**.

### 2. Orthogonalisation des Poids
Une fois cette direction identifiée, on modifie les matrices de poids du modèle (principalement les matrices de sortie des couches d'attention et MLP) pour les rendre **orthogonales** à ce vecteur.
* Mathématiquement, on "projette" les poids du modèle de manière à ce qu'ils ne puissent plus s'aligner sur la direction du refus.
* Le modèle conserve ses capacités de raisonnement technique mais perd sa capacité à déclencher un refus catégorique.



---

## Application dans ce Projet

Dans ce dépôt, la méthode a été adaptée pour répondre aux besoins spécifiques de la recherche en cybersécurité :

1.  **Cible** : Llama-3.2-3B-Instruct.
2.  **Modification** : Application de l'orthogonalisation sur les couches critiques (15 à 24) pour maximiser l'efficacité sans dégrader la cohérence du langage.
3.  **Résultat** : Un modèle capable de détailler des vulnérabilités critiques comme **PrintNightmare** sans message de refus moralisateur.

## Optimisation et Fluidité (Format GGUF)

Un défi majeur de ce projet était de faire tourner ce modèle de manière fluide sur un **Mac Intel**.
* **Conversion GGUF** : Le modèle abliteré a été converti du format Safetensors (6.4 Go) vers le format GGUF.
* **Quantification Q4_K_M** : Le modèle a été compressé en 4-bit (~2.1 Go). Cette étape est cruciale car elle permet au modèle de tenir entièrement dans la RAM, passant d'un temps de réponse de 30 minutes à une génération fluide en temps réel.

---

## Installation et Usage

### Prérequis
* [Ollama](https://ollama.com) installé sur macOS ou Linux.

### Téléchargement du Modèle

Le modèle est téléchargeable directement sur Hugging Face : **[[Cliquer pour télécharger le modèle](https://huggingface.co/rayanupc/Llama-3.2-3B-Cyber-Abliterated-GGUF/resolve/main/model-q4_k_m.gguf)]**

### Importation dans Ollama
Une fois le modèle téléchargé, clonez ce dépôt, placez le fichier `model-q4_k_m.gguf` dans le dossier et lancez la création du modèle :
```bash
ollama create llama3-cyber -f Modelfile

