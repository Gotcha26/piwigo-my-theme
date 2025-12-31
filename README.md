# piwigo-my-theme
Compilation d'actions pour la galerie publique

# En vrac

## Thème enfant
Création d'un thème enfant de **Bootstrap-Darkroom** nécessaire pour pouvoir modifier dans le dur certaines chsoes.
Inconvénients : La structure est alors "figée" par vous même et ne suivra pas les évolutions futur du thème parent sur les fichiers que vous aurez figées !

Conseil : Notez bien les modifications que vous faites pour mieux pouvoir les porter sur de futurs version ! 
Format : sous forme de patch. J'utilise Notepad++ avec le plugin "ComparePlus" afin d'exporter et vous présenter mes patchs. Ainsi, la compréhension en est plus aisée.

### Les templates

* Fichier ***bootstrap_darkroom_child/template/picture_info_cards.tpl***
```
--- picture_info_cards.tpl
+++ picture_info_cards.tpl
@@ -275,8 +275,8 @@
         </div>
       </div>
 {/if}
-      <div id="card-comments" class="ml-2">
+      <div id="card-comments" class="card ml-2">
         {include file='picture_info_comments.tpl'}
       </div>
     </div>
-{/if}
+{/if}
\ No newline at end of file
```

* Fichier ***bootstrap_darkroom_child/template/picture_info_cards.tpl***
```
--- Base.tpl
+++ Candidtat.tpl
@@ -1,78 +1,89 @@
-   <!-- comments -->
+<!-- comments -->
 {if isset($comment_add) || !empty($COMMENT_COUNT)}
-    <div id="comments">
-  {$shortname = $theme_config->comments_disqus_shortname}
-  {if $theme_config->comments_type == 'disqus' and !empty($shortname)}
-      <div id="disqus_thread"></div>
-{footer_script}{strip}
-var disqus_shortname = '{/strip}{$shortname}{strip}';
+  <div id="card-comment" class="card mb-2">
+    <div class="card-body">
+      <h5 class="card-title">{'Comments'|@translate}</h5>
 
-(function() {
-var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
-dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
-(document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
-})();
-{/strip}
-{/footer_script}
-  {else}
-      <ul class="nav nav-pills p-2" role="tablist">
-    {if !empty($COMMENT_COUNT)}
-        <li class="nav-item">
-          <a class="nav-link active" href="#viewcomments" data-toggle="pill" aria-controls="viewcomments">{$COMMENT_COUNT|@translate_dec:'%d comment':'%d comments'}</a>
-        </li>
-    {/if}
-    {if isset($comment_add)}
-        <li class="nav-item">
-          <a class="nav-link{if empty($COMMENT_COUNT)} active{/if}" href="#addcomment" data-toggle="pill" aria-controls="addcomment">{'Add a comment'|@translate}</a>
-        </li>
-    {/if}
-      </ul>
-      <div class="tab-content">
-    {if !empty($COMMENT_COUNT)}
-      <div id="viewcomments" class="tab-pane active">
-        {if isset($COMMENT_LIST)}
-          {$COMMENT_LIST}
+      <div id="comments">
+        {$shortname = $theme_config->comments_disqus_shortname}
+        {if $theme_config->comments_type == 'disqus' and !empty($shortname)}
+          <div id="disqus_thread"></div>
+          {footer_script}{strip}
+          var disqus_shortname = '{/strip}{$shortname}{strip}';
+
+          (function() {
+            var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
+            dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
+            (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
+          })();
+          {/strip}{/footer_script}
         {else}
-          {include file='comment_list.tpl'}
-        {/if}
-        {if !empty($navbar) }
-        <div class="row justify-content-center">
-          {include file='navigation_bar.tpl' fragment='comments'|@get_extent:'navbar'}
-        </div>
+          <ul class="nav nav-pills p-2" role="tablist">
+            {if !empty($COMMENT_COUNT)}
+              <li class="nav-item">
+                <a class="nav-link active" href="#viewcomments" data-toggle="pill" aria-controls="viewcomments">{$COMMENT_COUNT|@translate_dec:'%d comment':'%d comments'}</a>
+              </li>
+            {/if}
+            {if isset($comment_add)}
+              <li class="nav-item">
+                <a class="nav-link{if empty($COMMENT_COUNT)} active{/if}" href="#addcomment" data-toggle="pill" aria-controls="addcomment">{'Add a comment'|@translate}</a>
+              </li>
+            {/if}
+          </ul>
+
+          <div class="tab-content">
+            {if !empty($COMMENT_COUNT)}
+              <div id="viewcomments" class="tab-pane active">
+                {if isset($COMMENT_LIST)}
+                  {$COMMENT_LIST}
+                {else}
+                  {include file='comment_list.tpl'}
+                {/if}
+                {if !empty($navbar)}
+                  <div class="row justify-content-center">
+                    {include file='navigation_bar.tpl' fragment='comments'|@get_extent:'navbar'}
+                  </div>
+                {/if}
+              </div>
+            {/if}
+
+            {if isset($comment_add)}
+              <div id="addcomment" class="tab-pane{if empty($COMMENT_COUNT)} active{/if}">
+                <form method="post" action="{$comment_add.F_ACTION}">
+                  {if $comment_add.SHOW_AUTHOR}
+                    <div class="form-group">
+                      <label for="author">{'Author'|@translate}{if $comment_add.AUTHOR_MANDATORY} ({'mandatory'|@translate}){/if} :</label>
+                      <input class="form-control" type="text" name="author" id="author" value="{$comment_add.AUTHOR}">
+                    </div>
+                  {/if}
+
+                  {if $comment_add.SHOW_EMAIL}
+                    <div class="form-group">
+                      <label for="email">{'Email address'|@translate}{if $comment_add.EMAIL_MANDATORY} ({'mandatory'|@translate}){/if} :</label>
+                      <input class="form-control" type="text" name="email" id="email" value="{$comment_add.EMAIL}">
+                    </div>
+                  {/if}
+
+                  {if $comment_add.SHOW_WEBSITE}
+                    <div class="form-group">
+                      <label for="website_url">{'Website'|@translate} :</label>
+                      <input class="form-control" type="text" name="website_url" id="website_url" value="{$comment_add.WEBSITE_URL}">
+                    </div>
+                  {/if}
+
+                  <div class="form-group">
+                    <label for="contentid">{'Comment'|@translate} ({'mandatory'|@translate}) :</label>
+                    <textarea class="form-control" name="content" id="contentid" rows="5" cols="50">{$comment_add.CONTENT}</textarea>
+                  </div>
+
+                  <input type="hidden" name="key" value="{$comment_add.KEY}">
+                  <button type="submit" class="btn btn-primary btn-raised">{'Submit'|@translate}</button>
+                </form>
+              </div>
+            {/if}
+          </div>
         {/if}
       </div>
-    {/if}
-    {if isset($comment_add)}
-        <div id="addcomment" class="tab-pane{if empty($COMMENT_COUNT)} active{/if}">
-          <form method="post" action="{$comment_add.F_ACTION}">
-      {if $comment_add.SHOW_AUTHOR}
-            <div class="form-group">
-              <label for="author">{'Author'|@translate}{if $comment_add.AUTHOR_MANDATORY} ({'mandatory'|@translate}){/if} :</label>
-              <input class="form-control" type="text" name="author" id="author" value="{$comment_add.AUTHOR}">
-            </div>
-      {/if}
-      {if $comment_add.SHOW_EMAIL}
-            <div class="form-group">
-              <label for="email">{'Email address'|@translate}{if $comment_add.EMAIL_MANDATORY} ({'mandatory'|@translate}){/if} :</label>
-              <input class="form-control" type="text" name="email" id="email" value="{$comment_add.EMAIL}">
-            </div>
-      {/if}
-      {if $comment_add.SHOW_WEBSITE}
-            <div class="form-group">
-              <label for="website_url">{'Website'|@translate} :</label>
-              <input class="form-control" type="text" name="website_url" id="website_url" value="{$comment_add.WEBSITE_URL}">
-            </div>
-      {/if}
-            <div class="form-group">
-              <label for="contentid">{'Comment'|@translate} ({'mandatory'|@translate}) :</label>
-              <textarea class="form-control" name="content" id="contentid" rows="5" cols="50">{$comment_add.CONTENT}</textarea>
-            </div>
-            <input type="hidden" name="key" value="{$comment_add.KEY}">
-            <button type="submit" class="btn btn-primary btn-raised">{'Submit'|@translate}</button>
-          </form>
-        </div>
-      </div>
-    {/if}
     </div>
-  {/if}
+  </div>
 {/if}
```
Explications : C'est surtout l'agencement / imbrication qui a changé avec un ciblage plus précis pour les commentaires afin d'appliquer correctement le thème sur cette section qui apparaît mal d'origine.

### CSS

* Fichier ***bootstrap_darkroom_child/theme.css***
```
div#card-comments.card.ml-2 {
  margin-left: 0px !important;
}

.form-control {background-color: black;}
aria-controles.addcomment {
  font-size: 1.5rem; font-weight: 300;
}

 #show_exif_data:hover{
  color: #fff;
  background-color: #323232;
  border-color: #1d1d1d;
}
```

## Les paramètres globaux

Via le plugin *LocalFiles Editor*, dans la configuration local, ajouter ceci pour les exifs :

```
// +-----------------------------------------------------------------------+
// |                           LES META-DONNEES                            |
// +-----------------------------------------------------------------------+
 
//          CHAPITRE 1er
//                              LES CHAMPS EXIF
 
// Le visiteur pourra faire apparaître les méta-données EXIF sur picture.php
// en cliquant sur l'icône appropriée.
// show_exif: [false] - [true];
// Si vous choisissez "false" les champs ne seront pas affichés.
// Si vous choisissez "true" les champs seront affichés.
$conf['show_exif'] = true;
// FIXEME: supprimer l'icône en cas de paramètre à false.
 
// Piwigo peut stocker les informations EXIF dans la base de données.
// Cela facilite notamment les recherches. 
// Pour utiliser les métadonnées EXIF lors de la synchronisation:
// $conf['use_exif'] = [false] - [true];
// Si vous choisissez "false", les données ne seront pas enregistrées dans
// la BDD.
// Si vous choisissez "true", les données seront enregistrées dans la BDD.
$conf['use_exif'] = true;
 
// Si vous décidez d'enregistrer des champs EXIF dans la Base De Données,
// il faut dire "lesquels". Ce paramétrage est utilisé durant la 
// synchronisation. Chaque clé du tableau représente une colonne de la
// table images, chaque valeur correspond à un identifiant EXIF.
// Seuls les champs listés ci-après sont compatibles. N'en rajoutez pas plus
// ils ne seront pas inscrits dans la base de données.
$conf['use_exif_mapping'] = array(
  'date_creation' => 'DateTimeOriginal'
);
// Pour rajouter d'autres champs, il faut adapter votre Base De Données. (expérimental)

// Pour n'afficher que les champs EXIF nécessaires, il vous faut définir ici
// à l'avance quels champs seront à afficher
// (indépendant de use_exif_mapping).
// Évidement, les lignes vides n'apparaîtrons pas...
// Il est possible de choisir des champs parmi des groupes, par exemple
// ['COMPUTED']['ApertureFNumber']. Pour cela, créer une clé
// 'COMPUTED;ApertureFNumber'.
$conf['show_exif_fields'] = array(
  'Model',
  'UndefinedTag:0xA434',
  'FocalLength',
  'ExposureProgram',
  'ExposureTime',
  'FNumber',
  'ISOSpeedRatings',
  'ExposureBiasValue',
  'MeteringMode',
  'Flash',
  'ExposureMode',
  'WhiteBalance',
  'FileName',
  'DateTimeOriginal',
  'Software'
);
```
