# PFE_Radar_Wifibot
Ce projet met en place deux démonstrations de l'intégration du radar IWR6843, un radar produit par Texas Instruments et utilisant la technologie mmWave, avec un Wifibot. Le Wifibot utilise une carte NVIDIA Jetson Xavier NX en tant qu’ordinateur central. La carte NVIDIA utilise une version spéciale de Linux Ubuntu en tant que système d’exploitation, ce qui lui permet de supporter des ROS, un ensemble de logiciels open source qui servent de méta-système d’exploitation pour de nombreux robots. ROS est utilisé par ce projet pour intégrer le radar avec le robot et faciliter la visualisation du nuage de points récupéré par le radar. 

La première démonstration permet la visualisation du nuage de points dans le module RVIZ de ROS. La deuxième démonstration permet quant à elle l'arrêt du wifibot sur détection d'obstacle. Le wifibot avance en ligne droite jusqu'à ce que le radar détecte un objet devant lui. Cette démonstration utilise des versions modifiées de modules ROS pour le radar et le Wifibot ainsi qu’un module ROS générique afin d'implémenter le contrôle du robot et son arrêt devant les obstacles. Ce document explique comment utiliser le code fourni dans le git et le drive afin d’utiliser les deux démonstrations et tester l'intégration de la carte IWR6843ISK avec le Wifibot. Pour effectuer ces deux démonstrations il est nécessaire que le wifibot soit du même type que celui utilisé lors du PFE de l’ECE organisé avec Nexter Robotics et également que le radar exécute le lab ‘out-of-box’ et soit branché par câble USB à la carte NVIDIA.  

# Installation

ROS utilise le système Catkin pour la compilation du code en modules exécutables. Catkin utilise un dossier ‘workspace’ pour organiser la production de code, et dans le cas du Wifibot, ce dossier se trouve dans le dossier /mnt/nvme. 

Afin d’installer l’une ou l’autre des démos, il faut:

S’assurer que le dossier catkin_ws se trouve au bon endroit dans l’ordinateur du Wifibot, et que le dossier contient les dossiers devel, src, et build. Si ceci n’est pas le cas, suivre les instructions au lien suivant: http://wiki.ros.org/ROS/Tutorials/InstallingandConfiguringROSEnvironment en placant le workspace non pas  à ~ mais à /mnt/nvme. 

Extraire le zip/code ‘wifibot autonomous’ pour la démo d'arrêt sur détection d’obstacle, ou le zip/code ‘driver only’ pour la démo visualisation, et remplacer le dossier catkin_ws/src avec le dossier contenu dans le zip/dossier. 
Dans un terminal qui se trouve au niveau du dossier catkin_ws, exécuter la commande catkin make. Si il y a une erreure car le fichier wifibot_cmd.h ne peut pas être trouvé, copier les deux fichiers dans le dossier wifibot/code/fichiers Devels du drive ou git dans le dossier catkin_ws/devel/include/wifibot puis refaire la commande catkin make. 

Les modules nécessaires pour le fonctionnement des démos sont maintenant installés. Voir la prochaine partie pour l’utilisation des démonstrations. 

# Exécution

Avant de commencer, il faut s’assurer que le radar et le robot soient correctement connectés à l'ordinateur NVIDIA. En utilisant la commande ll /dev/seria/by-id, nous pouvons savoir à quels ports sont connectés le radar et le robot. Les démos supposent que le radar est connecté aux ports USB0 et USB1 tandis que le robot est connecté au port USB2. Le robot possède une connexion de type FTDI et le radar deux connexions de type Silicon-labs. Si les connexions ne sont pas dans le bon ordre, il suffit de débrancher le radar et le robot, puis rebrancher d'abord le radar puis le robot et revérifier. 

Pour la démonstration de visualisation, la visualisation peut être effectuée directement sur le robot (en branchant par câble HDMI ou DisplayPort la carte NVIDIA a un écran) ou sur un autre ordinateur qui se trouve sur le réseau wifi du wifibot (connexion ssh). 

Il faut simplement executer la commande suivante dans un terminal du wifibot:

	roslaunch ti_mmwave_roskpkg 6843_multi_3d_0bak.launch

Si l’on fait la visualisation depuis le Wifibot lui-même, on ouvre RVIZ en utilisant la commande rosrun rviz rviz dans un nouveau terminal. Ensuite on change le paramètre fixed frame (à gauche de la fenêtre RVIZ) à ti_mmwave_pcl et on ajoute (en utilisant le bouton add en bas à gauche de la fenêtre RVIZ un topic pointcloud2. On met le topic observé à radar_scan_pcl_0. Si tout se passe correctement, on devrait maintenant observer les points détectés par le radar sur l’écran. Les paramètres de visualisation du topic peuvent être modifiés afin de mieux percevoir les objets détectés.  

Si l’on fait cette démonstration en utilisant la visualisation depuis un ordinateur à distance, qui est sur le réseau du wifibot, il faut simplement utiliser ssh (ssh wifibot@<IP_DU_WIFIBOT>, mot de passe: wifibot) pour accéder au terminal de l’ordinateur NVIDIA du wifibot. Cependant pour la visualisation avec RVIZ depuis l’autre ordinateur il est nécessaire que les modules ROS du wifibot et de l’ordinateur puissent communiquer. Pour cela il faut tout d’abord s’assurer que ROS (version melodic de préférence) est bien installé sur l’ordinateur utilisé pour la visualisation à distance. De plus, il faut effectuer deux modifications. La première modification est d’ajouter les deux lignes suivantes au fichier ~/.bashrc du Wifibot avant de commencer la démonstration. 

  	export ROS_MASTER_URI=http://localhost:11311
  	export ROS_IP=<IP_DU_WIFIBOT>

L’IP du Wifibot devrait être 192.168.2.106. La deuxième modification à faire est d’ajouter les deux lignes suivantes au fichier ~/.bashrc du portable avant de commencer la démonstration. 


  	export ROS_MASTER_URI=http://<IP_DU_WIFIBOT>:11311
  	export ROS_IP=<IP_DE_L’ORDINATEUR_DE_VISUALISATION>

Grâce aux variables d'environnement mises en place avec ces deux modifications, le lancement de RVIZ sur l’ordinateur pourra communiquer avec les modules ROS du wifibot. 

Pour la démonstration d'arrêt du wifibot sur détection d'obstacle, il est également nécessaire de se connecter à l'ordinateur NVIDIA du Wifibot par ssh sur deux terminaux (ssh wifibot@<IP_DU_WIFIBOT>, mot de passe: wifibot). Dans le premier terminal il faut exécuter la commande suivante:

	roslaunch turtlebot_bringup minimal.launch 

Dans le deuxième terminal il faut exécuter la commande suivante :

	rosrun wifibot wifibot

Il est conseillé d'arrêter avec Ctrl+C l'exécution de la deuxième commande pour arrêter le robot si besoin, car arrêter la première commande n'empêche pas le robot de continuer d’avancer.  

Afin de modifier le comportement d’arrêt du robot lors de la détection d’un obstacle devant lui, il suffit de modifier le code du fichier /catkin_ws/src/ti_mmwave_rospkg/src/DataHandlerClass.cpp. 


