# Utilisation du Sélecteur de Langue CPCEC via CDN

Ce guide explique comment intégrer et utiliser la fonctionnalité de sélection de langue dans une application qui charge l'émulateur CPCEC via un CDN.

## Vue d'ensemble

L'émulateur CPCEC WebAssembly supporte maintenant la sélection de langue pour les ROMs du CPC 6128 :
- **Anglais** (`-le`) : ROM anglaise par défaut
- **Français** (`-lf`) : ROM française alternative

## Paramètre URL

L'émulateur détecte automatiquement la langue via le paramètre URL `lang` :

```bash
# Anglais (par défaut)
https://votredomaine.com/emulateur?lang=en

# Français
https://votredomaine.com/emulateur?lang=fr
```

## Configuration de l'émulateur

### Configuration de base du Module Emscripten

```javascript
// Configuration du Module Emscripten
var Module = {
    canvas: document.getElementById('canvas'),

    // Arguments pour la langue
    arguments: (function() {
        var args = [];
        var urlParams = new URLSearchParams(window.location.search);
        var lang = urlParams.get('lang');
        if (lang === 'fr') {
            args.push('-lf');  // ROM française
        } else {
            args.push('-le');  // ROM anglaise (défaut)
        }
        return args;
    })(),

    // Autres propriétés du Module...
    setStatus: function(text) {
        console.log('Status:', text);
    },

    print: function(text) {
        console.log('CPCEC:', text);
    }
};
```

## Interface utilisateur

### HTML pour le sélecteur de langue

```html
<div class="language-selector">
    <label for="lang-select">Language:</label>
    <select id="lang-select">
        <option value="en">English</option>
        <option value="fr">Français</option>
    </select>
</div>
```

### CSS pour styliser le sélecteur

```css
.language-selector {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    margin: 1rem 0;
    padding: 0.5rem;
    background: #f5f5f5;
    border-radius: 4px;
}

.language-selector label {
    font-weight: bold;
    color: #333;
}

.language-selector select {
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    background: white;
    cursor: pointer;
    font-size: 14px;
}

.language-selector select:focus {
    outline: none;
    border-color: #007bff;
    box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
}
```

## JavaScript - Gestion des événements

### Gestionnaire de changement de langue

```javascript
// Gestionnaire d'événement pour le sélecteur
document.getElementById('lang-select').addEventListener('change', function(e) {
    var lang = e.target.value;
    
    // Méthode recommandée : Recharger la page avec le nouveau paramètre
    var url = new URL(window.location);
    url.searchParams.set('lang', lang);
    window.location.href = url.toString();
});

// Initialisation de la valeur du sélecteur
document.addEventListener('DOMContentLoaded', function() {
    var urlParams = new URLSearchParams(window.location.search);
    var currentLang = urlParams.get('lang') || 'en';
    var selector = document.getElementById('lang-select');
    if (selector) {
        selector.value = currentLang;
    }
});
```

## Changement de langue programmatique

### Fonctions utilitaires

```javascript
/**
 * Change la langue de l'émulateur
 * @param {string} lang - 'en' ou 'fr'
 */
function setLanguage(lang) {
    if (lang !== 'en' && lang !== 'fr') {
        console.error('Langue non supportée:', lang);
        return;
    }
    
    // Mettre à jour l'URL et recharger
    var url = new URL(window.location);
    url.searchParams.set('lang', lang);
    window.location.href = url.toString();
}

/**
 * Obtient la langue actuelle
 * @returns {string} 'en' ou 'fr'
 */
function getCurrentLanguage() {
    var urlParams = new URLSearchParams(window.location.search);
    return urlParams.get('lang') || 'en';
}

/**
 * Bascule entre anglais et français
 */
function toggleLanguage() {
    var current = getCurrentLanguage();
    var next = current === 'en' ? 'fr' : 'en';
    setLanguage(next);
}
```

## Exemple d'intégration complète

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Émulateur CPC avec Sélection de Langue</title>
    <style>
        .language-selector {
            display: flex;
            align-items: center;
            gap: 0.5rem;
            margin: 1rem 0;
            padding: 0.5rem;
            background: #f5f5f5;
            border-radius: 4px;
        }
        
        .language-selector label {
            font-weight: bold;
            color: #333;
        }
        
        .language-selector select {
            padding: 0.5rem;
            border: 1px solid #ccc;
            border-radius: 4px;
            background: white;
            cursor: pointer;
        }
        
        #canvas {
            border: 1px solid #ccc;
            display: block;
            margin: 1rem 0;
        }
    </style>
</head>
<body>
    <h1>Émulateur Amstrad CPC</h1>
    
    <div class="controls">
        <div class="language-selector">
            <label for="lang-select">Language:</label>
            <select id="lang-select">
                <option value="en">English</option>
                <option value="fr">Français</option>
            </select>
        </div>
        
        <button id="load-btn">Charger un fichier</button>
        <button id="reset-btn">Reset</button>
    </div>
    
    <canvas id="canvas" width="768" height="544"></canvas>
    
    <script>
        // Configuration de l'émulateur
        var Module = {
            canvas: document.getElementById('canvas'),
            
            // Configuration de la langue via URL
            arguments: (function() {
                var urlParams = new URLSearchParams(window.location.search);
                var lang = urlParams.get('lang');
                return lang === 'fr' ? ['-lf'] : ['-le'];
            })(),
            
            // Gestion du statut
            setStatus: function(text) {
                console.log('Émulateur status:', text);
                document.getElementById('status').textContent = text;
            },
            
            // Gestion des messages
            print: function(text) {
                console.log('CPCEC:', text);
            },
            
            printErr: function(text) {
                console.error('CPCEC Error:', text);
            }
        };
        
        // Gestion du sélecteur de langue
        document.getElementById('lang-select').addEventListener('change', function(e) {
            var url = new URL(window.location);
            url.searchParams.set('lang', e.target.value);
            window.location.href = url.toString();
        });
        
        // Autres gestionnaires d'événements
        document.getElementById('reset-btn').addEventListener('click', function() {
            if (Module._em_reset) {
                Module._em_reset();
            }
        });
        
        // Initialisation
        document.addEventListener('DOMContentLoaded', function() {
            var urlParams = new URLSearchParams(window.location.search);
            var currentLang = urlParams.get('lang') || 'en';
            document.getElementById('lang-select').value = currentLang;
            
            // Créer un élément de statut si nécessaire
            if (!document.getElementById('status')) {
                var status = document.createElement('div');
                status.id = 'status';
                status.textContent = 'Chargement...';
                document.body.appendChild(status);
            }
        });
    </script>
    
    <!-- Charger l'émulateur depuis le CDN -->
    <script src="https://cdn.example.com/cpcec.js"></script>
</body>
</html>
```

## API JavaScript

### Propriétés du Module

- `arguments` : Tableau des arguments passés à l'émulateur
  - `['-le']` pour l'anglais
  - `['-lf']` pour le français

### Événements

- `change` sur `#lang-select` : Déclenche le rechargement avec la nouvelle langue

### Fonctions utilitaires recommandées

```javascript
// Fonctions à ajouter dans votre application
function setLanguage(lang) { /* ... */ }
function getCurrentLanguage() { /* ... */ }
function toggleLanguage() { /* ... */ }
```

## Dépannage

### Problèmes courants

1. **La langue ne change pas**
   - Vérifiez que le paramètre URL est correctement défini
   - Assurez-vous que l'émulateur supporte les arguments `-le`/`-lf`

2. **Le sélecteur ne s'initialise pas**
   - Vérifiez que le DOM est chargé avant d'accéder aux éléments
   - Utilisez `DOMContentLoaded` pour l'initialisation

3. **Erreur de chargement de l'émulateur**
   - Vérifiez l'URL du CDN
   - Assurez-vous que les fichiers `.wasm` et `.data` sont accessibles

### Debug

```javascript
// Afficher la langue actuelle
console.log('Langue actuelle:', getCurrentLanguage());

// Afficher les arguments passés à l'émulateur
console.log('Arguments émulateur:', Module.arguments);
```

## Support

Cette fonctionnalité est compatible avec :
- Navigateurs modernes supportant WebAssembly
- Emscripten Module API
- URLSearchParams API

Pour plus d'informations sur l'émulateur CPCEC, consultez la [documentation officielle](http://cngsoft.no-ip.org/cpcec.htm).
