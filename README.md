# methodology-income-housing
Méthodologie sous R pour ajouter la variable revenu à une population synthétique connaissant les déciles pour certaines modalités.

Nous avons testé deux méthodologies :
   * la première est décrite dans  heuristique_methodologie_revenue_menage.Rmd, cette méthodologie a été développée en premier. C'est celle qui a été mise en oeuvre sur l'exemple de Nantes
   * la seconde est décrite dans entropie_methodologie_revenue_menage.Rmd
   
  Finalement, la comparaison entre les deux méthodologies est décrit dans methodologie_revenue_menage.Rmd. C'est ce fichier qui est le résultat final du développement. Les autres fichiers, premiere_methodologie_revenue_menage.Rmd et deuxieme_methodologie_revenue_menage.Rmd, sont des fichiers qui sont en cours de développement.
  
  Le principe est de comparer les méthodologies sur un exemple simple que nous maitrisons. Pour cela nous commençons par générer une population synthétique.
  
 Nous générons une population synthétique avec deux variables qualitatives age de la personne de référence et taille du ménage.
 * age de la personne de reference a 3 niveaux : M_a=c('moins_de_30ans', '30-60ans','plus_de_60ans')
 * taille du ménage a 4 niveaux : M_t=c('1', '2','3','4_et_plus') # Modalite taille
 

 
 Il y a donc 12 modalités croisés. 
 Nous construisons un tableau de contingence qui croisent toutes les modalités des deux variables et qui contient pour chaque modalité croisée, sa proportion dans la population, le revenu moyen de cette proportion, l'écart type du revenu. A partir de ce tableau, nous générerons la population synthétique en utilisant pour chaque modalitée croisée une loi normale tronquée entre le revenu minimum, rmin, et le revenu maximum, rmax fixé à 10 fois rmin (soit 50000). 
 Le nombre de personnes dans la population synthétique est fixé à n
 Les proportions de chaque modalité croisée ont été fixées a priori. 
 Les revenus sont générés pour chaque modalité croisée à partir d'un revenu moyen et d'un écart type moyen fixés a priori. Attention, contrairement aux données filosofie qui fournissent un niveau de vie, ici, c'est bien un revenu ce qui veut dire qu'en moyenne un ménage comprenant deux individus a un revenue double d'un ménage comprenant un individu. Ceci est intéressant pour vérifier que les méthodologies vont bien affecter un revenue double au ménage qui sont plus nombreux.
 
 *Attention, nous avons fixé le noyau du générateur aléatoire (set.seed(1)) afin d'avoir tout le temps la même génération de revenu.*
 *Attention, le rmin et le rmax est le même pour toutes les modalités croisées, ce qui simplifie le nombre de degrés de liberté mais n'est pas le cas dans le cas nantais*
 
 Ensuite, nous construisons les déciles associés à chaque modalité de chaque variable qualitative.  Le nombre de decile est fixé a priori : nDec=2 => deux déciles entre 0 et 50% et entre 50% et 100%, ndec=10 => les déciles fournis par filosofie..
 
 
L'objectif de la méthodologie est de retrouver la variable renvenu à partir de la donnée de la population synthétique sans revenu d'une part et les déciles fournies pour chaque modalité non croisées. On se retrouve dans le même cas de figure qu'avec la base de données filosofie.


Deux méthodologies sont mises en oeuvre. Ells ont pour objectifs de trouver la probabilité d'un revenu connaissant les modalités croisés


La première est une méthodologie qui minimise l'entropie croisée entre la probabilité d'une modalité croisée connaissant le revenu et la probabilité de la mobilité croisée. Cette méthodologie effectue une minimisation par intervalle de revenu. Elle fait donc beaucoup d'optimisations mais chaque optimisation est légère. Le problème est que les solutions ne forment pas un ensemble cohérent de probabilité. Il y a donc une deuxième étape de mise en cohérence des probabilités. Par ailleurs, elle utilise une hypothèse forte sur la répartition uniforme des revenus dans chaque intervalle fournit pas filosofie. C'est la raison pour laquelle, nous l'appelons heuristique. Elle est relativiement rapide, donne une solution qu'il faut retraiter et qui n'est probablement pas globalement optimal.


La seconde est une méthodologie qui maximise l'entropie de la probabilite des modalités croisées et du revenu. Elle ne fait pas d'hypothèse sur la répartition des revenus et fournit des probabilités qui sont cohérentes. La contrepartie est que l'optimisation est complexe, c'est la raison pour laquelle, elle utilise trois algorithmes différents pour trouver une solution.

* Un simplexe est utilisé pour vérifier que les contraintes sont compatibles entres elles. Si les contraintes sont incompatibles, la solution est fournie par le simplexe. Dans le cadre étudié ici, le système est compatible car notre génération de données est cohérente. Dans le cas réel nantais, il y aura forcément des différences entre la base filosofie et la base insee : par exemple, le référent fiscal est différent du référent ménage....

* ensuite l'algorithme minxent.multiple est utilisé, il a donné entière satisfaction pour la première méthodologie, avec cette optimisation plus complexe, il sature des exponentielles qui deviennent inf. Ceci est notamment du à un Hessien qui a une chute de rang numérique. Lorsque ce problème est traité, la solution fournie ne respecte pas les contraintes et restent proches des probabilités uniformes....Bref, la solution fournie n'est pas du tout intéressante.

* enfin un sqp est mis en oeuvre qui donne un résultat compatible avec les contraintes.

La comparaison s'effectue avec un tableau fournissant pour les 12 modalités croisés
  * le revenu moyen et l'écart type de la gaussienne tronquée utilisée pour générer les revenus
  * le revenu moyen et l'écart type numérique de la population synthétique générer
  * le revenu moyen et l'écart type numérique de la loi des revenus trouvée avec la première méthodologie
  * le revenu moyen et l'écart type numérique de la loir des revenus trouvée avec la seconde méthodologie



  
