# MiniRT_RoadMap(WIP)
*Read this in other languages: [English](README.en.md)**(WIP)**, [Français](README.md)**(WIP)**.*
### GENERER LES RAYS
- Génerer le ray en fonction de la camera
  Premierement, le ray tracing, comme son nom l'indique consiste à tracer des rays d'une camera dans une direction données. Dans miniRT, votre camera est désignée par un point ( x, y, z ), un vecteur normalizé ( Vx, Vy, Vz ) et un FOV ( 0-180 ). Nous devons donc tracer des rays partant de son point, dans la direction du vecteur, tout en limitant le FOV.
 ![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/Camera.jpg)
- Pivoter le ray en fonction du pixel visé
- Limiter les rays en fonction du FOV
### INTERSECTION SPHERE
- Resoudre l'équation du second degré
- Calculer la normal
### INTERSECTION PLAN
- Vérifier l'intersection
- Calculer la normal
### INTERSECTION CYLINDER
- Résoudre l'équation du second degré (t1, t2)
- Calculer les caps de bout de cylindre (t3, t4)
- Calculer l'intersection
- Calculer la normal

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/DiffuseCylinder.png)
### LUMIERE AMBIENTE
- A quoi correspond la lumiere ambiente
- Simple calcul
### LUMIERE DIFFUSE
- A quoi correspond la lumiere diffuse
- Calcule de la lumiere diffuse
### OMBRES (HARD)
- Creation des rayons de lumière
- Calcul des ombres
### REDIMENSIONNER / PIVOTER / TRANSLATER
- Redimensionner (sphere: diametre / cylindre: Diametre, longueur)
- Pivoter (camera, plan, cylindre)
- Translater (camera, light, sphere, plan, cylindre)


# BONUS
### LUMIERE SPECULAIRE => PHONG REFLECTION MODEL
- A quoi correspond la lumiere speculaire
- Calcul de la lumiere speculaire

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/EverythingPhongLight.png)

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/PhongShadow.png)
### CHECKERBOARD
- Checkerboard sphere
- Checkerboard plan
- Checkerboard cylindre
- Checkerboard cone

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/Checkerboard.png)
### MULTIPLE-SPOT DE LUMIERE + LUMIERE COLORE
- Calculs d'addition des lumieres
- Calculs des mélanges de couleurs

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/MultipleSpot.png)
### INTERSECTION CONE
- Résoudre l'équation du second degré (t1, t2)
- Calculs des caps de haut et bas (t3, t4)
- Calcul de l'intersection
- Calcul de la normal

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/DiffuseConePlane.png)
### BUMB MAP
- A quoi correspond une normal map / Bump map
- Initialisation d'une bump map
- Exemple de calcul

![alt text](https://github.com/ZialeHub/MiniRT_RoadMap/blob/main/BumpMap.png)
