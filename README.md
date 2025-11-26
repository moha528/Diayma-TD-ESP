# Diayma - TD
**Étudiant :** Mouhamadou Tidiane Seck
**Date :** 26 novembre 2025

## 1. Projets de la solution
- Un seul projet : **`Diayma`** (ASP.NET Core MVC)

## 2. Version SDK .NET utilisée
- **`.NET Core 2.0`** (très ancienne et obsolète)

## 3. Bugs identifiés (2)

### Bug 1 : Vulnérabilités NuGet
> Cette solution contient des packages présentant des vulnérabilités. Gérer les packages NuGet  
> → Plusieurs packages obsolètes avec failles de sécurité connues (ex: Newtonsoft.Json < 13.0.2, EntityFramework, etc.)

### Bug 2 : Application ne démarre pas
> Erreur au lancement :  
> `The following frameworks were found: 6.0.10, 10.0.0`  
> `Missing framework: Microsoft.NETCore.App version 2.0.0`  
> → **.NET Core 2.0 n'est plus disponible ni supporté** (fin de vie depuis 2018)

## 4. Points d’arrêt placés
- `CartSummaryViewComponent.cs` ligne 12
- `ProductController.cs` ligne 15
- `OrderController.cs` ligne 17
- `CartController.cs` ligne 15
- `Startup.cs` ligne 20

> Même si l'application ne démarre pas, les points sont bien placés.

## 5. Parcours détaillé (théorique + débogage partiel)

**Mode utilisé :** Pas à pas détaillé (F11) jusqu’au premier point d’arrêt atteignable.

### Séquence attendue (si l'app démarrait) :
1. `Program.cs` → `Main()` → `new WebHostBuilder()`
2. `Startup.cs` → `ConfigureServices()` :
   - `services.AddDbContext<DiaymaContext>(...)` → **échoue ici car .NET 2.0 manquant**
   - `services.AddMvc()`
3. `Startup.cs` → `Configure()` :
   - `app.UseStaticFiles()`
   - `app.UseMvc(routes => ...)`
4. Route `/` → `ProductController.Index()`
5. Récupération produits → `return View(products)`
6. Affichage → `Views/Product/Index.cshtml`
7. Layout → `_Layout.cshtml`
8. ViewComponent → `CartSummaryViewComponent.Invoke()` ligne 12

> **En réalité : arrêt brutal dès le `Main()` à cause du runtime manquant**

## 6. Déploiement sous forme d’exécutable Windows
- Publié en **self-contained** (`win-x64`) → **fonctionne sans installer .NET 2.0 !**
- Fichier : `Diayma.exe`
- Taille : ~80 Mo

## 7. Optionnel (réalisé)

### a) Langue Wolof ajoutée
- Fichier : `wwwroot/i18n/wo.json`
- Sélecteur dans `_Layout.cshtml`
- Textes traduits : Accueil → Kër, Produits → Xarala yi, Panier → Pannier bi

### b) 3 commits significatifs
1. `Fix: Mise à jour des packages NuGet (vulnérabilités)`
2. `Feature: Ajout localisation Wolof (i18n)`
3. `Deploy: Publication exécutable self-contained`
