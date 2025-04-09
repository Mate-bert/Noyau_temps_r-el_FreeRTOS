# Noyau_temps_reel_FreeRTOS

##01. Premier pas

1) le fichier main.c se sittue ici : Core\Src\main.c

2) Ce sont les zones qui vont être protégé lors de chaque régénération du code, il est donc fortement conseillé d'écrire entre le BEGIN et le END sous peine de perdre sont code si l'on regénère le projet.

3) Dans la fonction HAL_GPIO_TogglePin on doit passer en paramètre Le type de GPIO qu'on veut utiliser dans notre cas "GPIOI" ainsi que la PIN qu'on utilise pour nous c'est "GPIO_PIN_1". Et dans le HAL_Delay, on passe en paramètre le temps en ms qu'on veut entre chaque itération du while, nous avons choisit 500ms dans notre code.

4) Les entrées/sorties sont définie dans le fichier Core\Src\gpio.c .

5) Pour faire clignoter la LED on écrit ce programme dans le while (1)
   HAL_GPIO_TogglePin(GPIOI, GPIO_PIN_1);
   HAL_Delay(500);

6) Pour ajouter la fonctionnalité d'allumage de la LED avec l'appuie du boutton, on modifie le code dans le while(1) par ça :
  if (HAL_GPIO_ReadPin(GPIOI, GPIO_PIN_11) == GPIO_PIN_SET)
		{
		  HAL_GPIO_WritePin(GPIOI, GPIO_PIN_1, GPIO_PIN_RESET);
		}
  else
	  {
		  HAL_GPIO_WritePin(GPIOI, GPIO_PIN_1, GPIO_PIN_SET);
	  }
  HAL_Delay(50);

 ## 1. FreeRTOS, Tâches et semaphores

 ### 1.1 tache simple

 1) TOTAL_HEAP_SIZE défini l'espace que l'on accorde au Kernel de FreeRTOS sur la RAM. Lorsque on change ce paramètre, un define vois sa valeur changer dans FreeRTOSConfig.h.

 2) TICK_PERIOD_MS permet de définir le temps d'attente entre chaque exécution de la fonction.

### 1.3 Notification



