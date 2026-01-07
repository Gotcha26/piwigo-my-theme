# Babar

* Téléchargement : [PEM](https://fr.piwigo.org/ext/index.php?eid=1040)

Sous le nom assez curiez, se cache une fonction bien pratique et originale qui permet de définir une bannière pour l'album centrée sur un point précis.  
La bannière est ainsi très étroite et vient se positionner dans l'entête de la page mais ça donne un coté personnalisation qui vient rompre l'aspect solennel et dépouillé de ma galerie.

## Configuration

Rien ! Nada.
<img width="1195" height="859" alt="Screenshot 2026-01-07 at 15-57-15 Julien Moreau gallery&#39;s Administration de Piwigo" src="https://github.com/user-attachments/assets/699e6137-aa3e-43b7-b47a-fcb497ee66ff" />

## Utilisation

* Se rendre sur la page d'administration de l'album.
* Onglet : "Babar"
* Faire correspondre le cadre de séléction avec l'élement à centrer sur votre bannière.
<img width="1513" height="1711" alt="Screenshot 2026-01-07 at 16-29-57 Julien Moreau gallery&#39;s Administration de Piwigo" src="https://github.com/user-attachments/assets/60b07250-af9d-4a49-b807-cd392bc150b2" />



## Hacks

Le fonctionnement de base n'était pas optimal alors j'ai bricolé le code.  

> [!NOTE]
> Les hacks sont basés sur la version **1.8**.

Source : [Claude.ia](https://claude.ai/chat/9646138a-81a5-4fa3-8451-072aa19a16ef)

* Fichier [events_public.inc.php](#events_public.inc.php)
* Fichier [maintain.class.php](#maintain.class.php)
* Fichier [album_hoi.php](#album_hoi.php) 

---

* Fichier : _include/events_public.inc.php_
<a name="events_public.inc.php"></a> 
```
--- include\events_public.inc.php
+++ pid-19480\events_public.inc.php
@@ -2,0 +3,18 @@
+/**
+ * Récupère le thème effectif (parent si c'est un thème enfant)
+ */
+function babar_get_theme_name($user_theme) {
+  global $conf;
+  
+  $theme_path = PHPWG_ROOT_PATH.'themes/'.$user_theme.'/themeconf.inc.php';
+  
+  if (file_exists($theme_path)) {
+    include($theme_path);
+    if (isset($themeconf['parent']) && !empty($themeconf['parent'])) {
+      return $themeconf['parent'];
+    }
+  }
+  
+  return $user_theme;
+}
+
@@ -9,0 +28,2 @@
+  $effective_theme = babar_get_theme_name($user['theme']);
+
@@ -17,1 +37,1 @@
-    if ($user['theme'] == 'bootstrap_darkroom') {
+    if (str_starts_with($effective_theme, 'bootstrap_darkroom')) {
@@ -68,2 +88,1 @@
-      $decalage = '-'. $result[0]['decalage'];
-
+      $decalage = $result[0]['decalage']; // Pourcentage stocké en BDD
@@ -71,1 +90,1 @@
-      $decalage = 'center-';
+      $decalage = 50; // Centre par défaut (50%)
@@ -113,1 +132,1 @@
-    if ($user['theme'] == 'bootstrap_darkroom') {
+     if (str_starts_with($effective_theme, 'bootstrap_darkroom')) {
@@ -123,102 +141,0 @@
-// BOOTSTRAP DARKROOM
-// +-----------------------------------------------------------------------+
-/*function babar_loc_begin_page_header_darkroom()
-{
-
-  global $template, $page, $conf, $user;
-  
-  //V1.5 : them bootstrap darkroom
-  if ($user['theme'] == 'bootstrap_darkroom') {
-    // V1.1
-    if ( isset($page['is_homepage']) and $page['is_homepage'] and $conf['babar']['change_homepage']) {
-      // si existe default_picture, alors yatil un decalage ?
-      if ($conf['babar']['default_picture'] !== "") {
-        $todo = "pas facile";
-      }
-      $template->assign('DMA_THEME', $conf['babar']['dma_theme']);
-      $template->assign('H_BANNIERE', $conf['babar']['hauteur_banniere'] .'px');
-      $template->assign('CHANGE_HOMEPAGE', $conf['babar']['change_homepage']);
-      $template->assign('DEFAULT_PICTURE', $conf['babar']['default_picture']);
-      $template->assign('babar_page_header_full', $conf['babar']['page_header_full']);
-
-      $template->set_prefilter('header', 'babar_loc_begin_page_header_prefilter_homepage');
-
-    }
-
-    // V1.1: on ne s'occupe plus du page header du theme
-    if ( isset($page['category'])
-        and (isset($page['category']['representative_picture_id'])
-              or $conf['babar']['use_default'])) {
-
-      // v1.2 : si pas de representative, prendre image par defaut
-      if ( isset($page['category']['representative_picture_id'])) {
-        $representative = $page['category']['representative_picture_id'];
-        $query = 'SELECT i.id, i.path, i.file, c.babar as decalage FROM ' . IMAGES_TABLE .' i 
-          INNER JOIN ' . CATEGORIES_TABLE . ' c ON i.id = c.representative_picture_id 
-          WHERE c.representative_picture_id = ' . $representative . ';';
-      } else {
-        $pos = strrpos($conf['babar']['default_picture'], '/');
-        $defaultfile = substr($conf['babar']['default_picture'], $pos+1);
-        $query = 'select id from ' . IMAGES_TABLE . ' where path like ("%/'. $defaultfile .'");';
-        $result = query2array($query);
-        $representative  = $result[0]['id'];
-        $query = 'SELECT i.id, i.path, i.file, null as decalage FROM ' . IMAGES_TABLE .' i 
-          WHERE i.id = ' . $representative . ';';
-      }
-      $result = query2array($query);
-      
-      // V1.4 l'image par defaut existe-t-elle ?
-      // verifier avant, coté admin
-
-      // V1.1 : l'original peut ne pas etre autorisé, travailler avec la derivative-xx
-      @$derivative = DerivativeImage::get_one(IMG_XXLARGE, $result[0])->get_path();
-
-      // si la miniature n'existe pas
-      if (!is_file($derivative)) {
-        // on crée le src avec i.php qui genere le format choisi
-        $pos = strrpos($result[0]['path'], '.' );
-        $ext = get_extension($result[0]['file']);
-        $src = "i.php?" . substr ( $result[0]['path'], 1, $pos - 1 ) . "-xx." . $ext;
-      } else {
-        // sinon j'utilise l'existant
-        $src = $derivative;
-      }
-
-      $url_representative = 'background-image: url('. $src .')';
-      if ($result[0]['decalage'] != null) {
-        $decalage = '-'. $result[0]['decalage'];
-
-      } else {
-        $decalage = 'center-';
-      }
-
-      // description selon comment et/ou option babar show_name de  l'album
-      $thecomment = "";
-      if ($conf['babar']['show_name']) {
-        // background-color: rgba(0,0,0,0.2);
-        $thecomment .= "<span style='color: currentColor;font-size: 5vh;'>".$page['category']['name']."</span><br>";
-      }
-
-      // v1.4 : si show_descrition, displaycontent true sinon false
-      if (!is_null($page['category']['comment']) and true == $conf['babar']['show_description']) {
-        $thecomment .= $page['category']['comment'];
-        $description_on_banner = true;
-      } else {
-        $description_on_banner = false;
-      }
-    
-      $template->assign('H_BANNIERE', $conf['babar']['hauteur_banniere'] .'px');
-      $template->assign('DMA_THEME', $conf['babar']['dma_theme']);
-      $template->assign('CHANGE_HOMEPAGE', $conf['babar']['change_homepage']);
-      $template->assign('URL_REPRESENTATIVE', $url_representative);
-      $template->assign('CAT_ID', $page['category']['id']);
-      $template->assign('COMMENT', $thecomment);
-      $template->assign('DECALAGE', $decalage);
-      $template->assign('description_on_banner', $description_on_banner);
-      // V1.4 : fin
-
-      $template->set_prefilter('header', 'babar_loc_begin_page_header_prefilter');
-    }
-  }
-}*/
-
@@ -237,1 +154,2 @@
-  $replaceBD = '<!-- DMA : Babar Darkroom Homepage-->
+  $replaceBD = '
+  <!-- DMA : Babar Darkroom Homepage-->
@@ -272,4 +189,0 @@
-  /*$search2 = '<nav class="navbar navbar-expand-lg navbar-main {$theme_config->navbar_main_bg} {if $theme_config->page_header == \'fancy\'}navbar-dark navbar-transparent fixed-top{else}{$theme_config->navbar_main_style}{/if}">';
-  $replace2 = '<nav class="navbar navbar-expand-lg navbar-main {$theme_config->navbar_main_bg} navbar-dark navbar-transparent fixed-top">';
-  $retour2 = str_replace($search2, $replace2 , $retour1);*/
-
@@ -287,1 +201,2 @@
-  $replaceBD = '<!-- DMA Babar Darkroom Albums -->
+  $replaceBD = '
+  <!-- DMA Babar Darkroom Albums -->
@@ -300,42 +215,1 @@
-      {if $DECALAGE != \'center-\'}
-        @media (min-width: 576px) {
-          .page-header-image {
-            background-position-y: calc({$DECALAGE}px * 0.4) !important;
-          }
-        }
-        @media (min-width: 768px) {
-          .page-header-image {
-            background-position-y: calc({$DECALAGE}px * 0.6) !important;
-          }
-        }
-        @media (min-width: 992px) {
-          .page-header-image {
-            background-position-y: calc({$DECALAGE}px * 0.8) !important;
-          }
-        }
-        @media (min-width: 1200px) {
-          .page-header-image {
-            background-position-y: {$DECALAGE}px !important;
-          }
-        }
-        @media (min-width: 1408px) {
-          .page-header-image {
-            background-position-y: calc({$DECALAGE}px * 1.3) !important;
-          }
-        }
-        @media (min-width: 1536px) {
-          .page-header-image {
-            background-position-y: calc({$DECALAGE}px * 1.7) !important;
-          }
-        }
-        @media (min-width: 1920px) {
-          .page-header-image {
-            background-position-y: calc({$DECALAGE}px * 2) !important;
-          }
-        }
-        @media (min-width: 2240px) {
-          .page-header-image {
-            background-position-y: calc({$DECALAGE}px * 3) !important;
-          }
-        }
-        @media (min-width: 2560px) {
+      {if $DECALAGE != 50}
@@ -343,2 +217,1 @@
-            background-position-y: calc({$DECALAGE}px * 3.9) !important;
-          }
+          background-position: center {$DECALAGE}% !important;
@@ -346,1 +219,1 @@
-        @media (min-width: 3200px) {
+      {else}
@@ -348,2 +221,1 @@
-            background-position-y: calc({$DECALAGE}px * 4) !important;
-          }
+          background-position: center center !important;
@@ -374,8 +245,0 @@
-  // si autre que bootstrap darkroom : yapa de jumbotron hors darkroom
-  /*
-  $replaceM = '<!-- tests DMA M-->';
-
-  global $user;
-  $toreplace = $replaceM;
-  if ($user['theme'] == 'bootstrap_darkroom'){
-	*/
@@ -383,1 +246,0 @@
-	//}
@@ -391,8 +253,0 @@
-  /*$un = substr($retour,0, 1000);
-  $deux = substr($retour, 1000, 1000);
-  $trois = substr($retour, 2000, 1000);
-  $quetre = substr($retour, 3000, 1000);
-  $cinq = substr($retour, 4000, 1000);
-  $six = substr($retour, 5000, 1000);
-  $sept = substr($retour,6000, 1000);*/
-
@@ -403,103 +257,0 @@
-// MODUS
-// +-----------------------------------------------------------------------+ 
-/*function babar_loc_begin_page_menubar()
-{
-  global $template, $conf, $user, $page;
-  
-  if ($user['theme'] != 'bootstrap_darkroom') {
-
-    // Babar Modus Homepage
-    if ( isset($page['is_homepage']) and $page['is_homepage'] and $conf['babar']['change_homepage']) {
-      $template->assign('DMA_THEME', $conf['babar']['dma_theme']);
-      $template->assign('H_BANNIERE', $conf['babar']['hauteur_banniere'] .'px');
-      $template->assign('CHANGE_HOMEPAGE', $conf['babar']['change_homepage']);
-      $template->assign('DEFAULT_PICTURE', $conf['babar']['default_picture']);
-      $template->assign('babar_page_header_full', $conf['babar']['page_header_full']);
-    // V1.5 : Modus, le style dans style.css
-    $template->func_combine_css(array('path' => 'plugins/babar/admin/template/style.css'));
-
-      $template->set_prefilter('menubar', 'babar_loc_begin_page_menubar_prefilter_homepage');
-
-    }
-
-    if ( isset($page['category'])
-        and (isset($page['category']['representative_picture_id'])
-              or $conf['babar']['use_default'])
-        and (!isset($page['image_id']) or $page['image_id'] == 0)) {
-
-      // v1.2 : si pas de representative, prendre image par defaut
-      if ( isset($page['category']['representative_picture_id'])) {
-        $representative = $page['category']['representative_picture_id'];
-        $query = 'SELECT i.id, i.path, i.file, c.babar as decalage FROM ' . IMAGES_TABLE .' i 
-          INNER JOIN ' . CATEGORIES_TABLE . ' c ON i.id = c.representative_picture_id 
-          WHERE c.representative_picture_id = ' . $representative . ';';
-      } else {
-        $pos = strrpos($conf['babar']['default_picture'], '/');
-        $defaultfile = substr($conf['babar']['default_picture'], $pos+1);
-        $query = 'select id from ' . IMAGES_TABLE . ' where path like ("%/'. $defaultfile .'");';
-        $result = query2array($query);
-        $representative  = $result[0]['id'];
-        $query = 'SELECT i.id, i.path, i.file, null as decalage FROM ' . IMAGES_TABLE .' i 
-        WHERE i.id = ' . $representative . ';';
-      }
-      $result = query2array($query);
-
-      // V1.4 l'image par defaut existe-t-elle ?
-      // verifier avant, coté admin
-
-      // V1.1 : l'original peut ne pas etre autorisé, travailler avec la derivative-xx
-      @$derivative = DerivativeImage::get_one(IMG_XXLARGE, $result[0])->get_path();
-
-      // si la miniature n'existe pas
-      if (!is_file($derivative)) {
-        // on crée le src avec i.php qui genere le format choisi
-        // PB : si resolution trop faible, redirection sur l'originale
-        $pos = strrpos($result[0]['path'], '.' );
-        $ext = get_extension($result[0]['file']);
-        $src = "i.php?" . substr ( $result[0]['path'], 1, $pos - 1 ) . "-xx." . $ext;
-      } else {
-        // sinon j'utilise l'existant
-        $src = $derivative;
-    }
-
-    $url_representative = 'background-image: url('. $src .')';
-    if ($result[0]['decalage'] != null) {
-      $decalage = '-'. $result[0]['decalage'];
-
-    } else {
-      $decalage = 'center-';
-    }
-
-    // description selon comment et/ou option babar show_name de  l'album
-    $thecomment = "";
-    if ($conf['babar']['show_name']) {
-      // background-color: rgba(0,0,0,0.2);
-      $thecomment .= "<span style='color: currentColor;font-size: 5vh;'>".$page['category']['name']."</span><br>";
-    }
-
-    // v1.4 : si show_descrition, displaycontent true sinon false
-    if (!is_null($page['category']['comment']) and true == $conf['babar']['show_description']) {
-      $thecomment .= $page['category']['comment'];
-      $description_on_banner = true;
-    } else {
-      $description_on_banner = false;
-    }
-
-    $template->assign('H_BANNIERE', $conf['babar']['hauteur_banniere'] .'px');
-    $template->assign('DMA_THEME', $conf['babar']['dma_theme']);
-    $template->assign('CHANGE_HOMEPAGE', $conf['babar']['change_homepage']);
-    $template->assign('URL_REPRESENTATIVE', $url_representative);
-    $template->assign('CAT_ID', $page['category']['id']);
-    $template->assign('COMMENT', $thecomment);
-    $template->assign('DECALAGE', $decalage);
-    $template->assign('description_on_banner', $description_on_banner);
-    // V1.4 : fin
-    // V1.5 : Modus, le style dans style.css
-    $template->func_combine_css(array('path' => 'plugins/babar/admin/template/style.css'));
-
-
-    $template->set_prefilter('menubar', 'babar_loc_begin_page_menubar_prefilter');
-    }
-  }
-}*/
-
@@ -513,1 +265,2 @@
-  $toreplaceM = '<!-- DMA : Babar Modus Homepage -->
+  $toreplaceM = '
+  <!-- DMA : Babar Modus Homepage -->
@@ -554,1 +307,2 @@
-  $replaceM = '<!-- DMA : Babar Modus Albums -->
+  $replaceM = '
+  <!-- DMA : Babar Modus Albums -->
@@ -564,42 +318,1 @@
-        {if $DECALAGE != \'center-\'}
-          @media (min-width: 576px) {
-            .page-header-image {
-              background-position-y: calc({$DECALAGE}px * 0.4) !important;
-            }
-          }
-          @media (min-width: 768px) {
-            .page-header-image {
-              background-position-y: calc({$DECALAGE}px * 0.6) !important;
-            }
-          }
-          @media (min-width: 992px) {
-            .page-header-image {
-              background-position-y: calc({$DECALAGE}px * 0.8) !important;
-            }
-          }
-          @media (min-width: 1200px) {
-            .page-header-image {
-              background-position-y: {$DECALAGE}px !important;
-            }
-          }
-          @media (min-width: 1408px) {
-            .page-header-image {
-              background-position-y: calc({$DECALAGE}px * 1.3) !important;
-            }
-          }
-          @media (min-width: 1536px) {
-            .page-header-image {
-              background-position-y: calc({$DECALAGE}px * 1.7) !important;
-            }
-          }
-          @media (min-width: 1920px) {
-            .page-header-image {
-              background-position-y: calc({$DECALAGE}px * 2) !important;
-            }
-          }
-          @media (min-width: 2240px) {
-            .page-header-image {
-              background-position-y: calc({$DECALAGE}px * 3) !important;
-            }
-          }
-          @media (min-width: 2560px) {
+        {if $DECALAGE != 50}
@@ -607,2 +320,1 @@
-              background-position-y: calc({$DECALAGE}px * 3.9) !important;
-            }
+            background-position: center {$DECALAGE}% !important;
@@ -610,1 +322,1 @@
-          @media (min-width: 3200px) {
+        {else}
@@ -612,2 +324,1 @@
-              background-position-y: calc({$DECALAGE}px * 4) !important;
-            }
+            background-position: center center !important;
```

* Fichier : _maintain.class.php_
<a name="maintain.class.php"></a> 
```
--- maintain.class.php
+++ maintain.class-1.php
@@ -57,1 +57,1 @@
-      pwg_query('ALTER TABLE `' . CATEGORIES_TABLE . '` ADD `babar` smallint(5);');
+      pwg_query('ALTER TABLE `' . CATEGORIES_TABLE . '` ADD `babar` DECIMAL(5,2);');
```

* Fichier : _admin/album_hoi.php_
<a name="album_hoi.php"></a> 
```
--- admin\album_hoi.php
+++ Temp\album_hoi.php
@@ -43,6 +42,0 @@
-    /* Deplacé dans config
-    // sauvegarder la hauteur de banniere, max à 350px
-    conf_update_param('babar_hauteur_banniere', min(350, $_POST['banner_height']));
-    $conf['babar']['hauteur_banniere'] = min(350, $_POST['banner_height']);
-    */
-
@@ -60,4 +54,4 @@
-    // calculer decalage
-    $ycible = $_POST['y'];                              // nouvelle limite haute de la banniere 
-    $realdecalage = $_POST['y2'] - $_POST['y'];         // hauteur reel de la banniere
-    $decalage = round($conf['babar']['hauteur_banniere'] * $_POST['y'] / $realdecalage);    // decalage en px à appliquer
+    // calculer le centre vertical de la sélection
+    $center_y = ($_POST['y'] + $_POST['y2']) / 2;
+    // calculer le décalage en pourcentage par rapport à la hauteur totale de l'image
+    $decalage_percent = ($center_y / $_POST['height']) * 100;
@@ -65,7 +59,1 @@
-    // memoriser ce decalage dans table categories.babar
-    pwg_query('UPDATE ' . CATEGORIES_TABLE . ' SET babar = '.$decalage.' WHERE id = '.$cat_id.';');
-
-    // V1.6 : memoriser le decalage smallint(5) avec l'image pour utilisation si image par defaut
-    //pwg_query('UPDATE ' . IMAGES_TABLE . ' SET babar = '.$decalage.' WHERE id = '.$_POST['picture_id'].';');
-
-    $page['infos'][] = l10n('Your configuration settings are saved')." ".$decalage;
+    pwg_query('UPDATE ' . CATEGORIES_TABLE . ' SET babar = '.$decalage_percent.' WHERE id = '.$cat_id.';');
```

