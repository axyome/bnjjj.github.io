---
layout: posts
title:  "Bien structurer ses tâches Gulp"
subtitle: "Diverses optimisations pour vos tâches gulp."
author: "Bnjjj (Benjamin Coenen)"
avatar: "/img/authors/bnjjj.jpeg"
image: "/img/gulp-logo.jpg"
date:   2015-12-05 19:00:00
public: true
anchor: structure-gulp
permalink: /structure-gulp
comments: true
---

Bonjour à tous, aujourd'hui je vous propose un petit article sur la mise en place d'une structure de gulpfile pour gulp afin d'optimiser sa maintenabilité et sa rapidité. Je ne vais pas expliquer le fonctionnement de gulp ni en quoi il est utile, la connaissance de gulp est donc un prérequis pour la lecture de cet article.

La question principale est : pourquoi avoir une bonne structure pour nos gulpfiles ? En fait, il n'y a que peu de temps que je me suis vraiment rendu compte de l'importance à optimiser au maximum le temps d'exécution de ces tâches automatiques, de build, de lint, etc... Car quand on travaille sur de petits projets perso avec peu de codes et process mis en place tout va bien. Mais une fois qu'on arrive avec un projet qui grandit et contient énormément de tâches et d'options, on a intérêt à optimiser ces systèmes d'automatisation avant de devenir fou à attendre chaque rebuild du système. Evidemment quand on développe ou débug on n'a pas envie d'attendre à chaque fois 30 secondes que les tâches se terminent. Rien que sur une journée c'est une perte de temps considérable alors ne parlons même pas du temps perdu sur un projet entier.

C'est pour ces raisons que je me suis un peu attardé sur le problème et j'ai vraiment essayé de me faire un système modulaire, rapide tout en restant flexible. Attention, je n'ai pas la prétention de dire que la solution que je vous propose est la meilleure. Au contraire, écrire un article pour me permettre d'échanger à ce propos et trouver une meilleure solution est sûrement une de mes principales motivations. 

Commençons par la structure des fichiers. J'ai décidé de séparer chaque tâche dans un fichier séparé et de placer tous ces fichiers dans un dossier nommé "_gulp-scripts_". Il y a plusieurs avantages à organiser nos fichiers de cette manière. Puisque nos fichiers porteront le même nom que la tâche qui la contient on trouvera plus facilement notre tâche et si on ne l'utilise plus il suffit tout simplement de supprimer le fichier.
![tree](/img/tree-gulpfile.jpg){: .center-image }

On peut très bien aussi imaginer placer ces fichiers de tâches encore dans d'autres sous-dossiers en fonction de l'environnement de build (dev, prod, ...). Mais pour faciliter l'explication je ne vais pas le mettre en application dans cet exemple. Notre gulpfile.js placé à la racine de notre projet va simplement nous servir pour importer ("_require_") toutes les tasks que l'on a créées dans le dossier "_gulp-scripts_". Il n'y a pas plus simple comme gulpfile principal.

Voici le contenu de mon gulpfile principal : 

---

{% highlight javascript %}
var gulp = require('gulp'),
  requireDir = require('require-dir');

requireDir('./gulp-scripts', {recurse: true});
{% endhighlight %}

---

Pour une tâche et donc un fichier il y a sa tâche et la tâche en watch associée. Celle que j'appelle watch est celle qui sera doté d'un watcher sur les fichiers concernés. C'est à dire que dès que l'on va changer ces fichiers ça relancera la tâche associée.

---

{% highlight javascript %}

'use strict';
var gulp = require('gulp');
var babel = require('gulp-babel');
 
gulp.task('babel', function () {
  return gulp.src('./www')
    .pipe(babel())
    .pipe(gulp.dest('./dist/es6'));
});

gulp.task('babel:watch', function () {
  gulp.watch('./www', ['babel']);
});

{% endhighlight %}

---

Grâce à ces deux distinctions on va pouvoir créer des tâches personnalisées qui lient les tâches entre elles et on appellera leur task watch ("_nomDeLaTache:watch_") à l'intérieur. De cette façon lorsqu'on modifiera un fichier watché on ne va relancer que les tâches concernées. Grâce à ça nos tâches ne se relanceront que si elles sont concernées, et les autres ne seront pas relancées pour rien. Et ça c'est un point TRÈS important pour notre optimisation de temps et process.

Voici un exemple si je modifie un fichier JavaScript : ![wath-js](/img/jswatch.jpg){: .center-image-large }

Voici un exemple si je modifie un fichier HTML : ![wath-js](/img/watchhtml.jpg){: .center-image-large }

On peut constater que mon gulp ne se lance que pour les fichiers concernés.

Une autre optimisation bien utile aussi est de sécuriser nos streams de tasks gulp. En effet, imaginons que j'ai une tâche sass qui transpile mon sass en css. Si je fais une erreur de syntaxe et enregistre notre tâche va planter et donc gulp va se couper brusquement. Encore une fois si on trouve directement d'où vient le problème ce n'est pas gênant mais si ce n'est pas le cas on est bon à relancer gulp à chaque crash. Pour éviter ce genre de soucis je vous conseille d'utiliser "_[gulp-plumber](https://www.npmjs.com/package/gulp-plumber)_" qui va pemettre de ne pas couper notre gulp s'il y a une erreur.

Pour conclure si vous souhaitez tester assez rapidement et facilement cette structure, je vous invite à télécharger le générateur yeoman __[Gulpfile-advanced](https://www.npmjs.com/package/generator-gulpfile-advanced)__ que j'ai développé avec la même structure que je viens de vous présenter. N'hésitez pas aussi à développer vos propres tasks et sous générateurs (cf : __[Contributing.md](https://github.com/bnjjj/generator-gulpfile-advanced#contributing)__) et me les proposer via __[Github](https://github.com/bnjjj/generator-gulpfile-advanced)__ (pull-requests). Chacun de vos sous générateurs sera ainsi intégré sous forme de submodule et donc vous resterez mainteneur et propriétaire de votre code.

Voilà, en espérant que cet article vous sera utile et encore une fois n'hésitez pas à me faire vos retours dans les commentaires.
