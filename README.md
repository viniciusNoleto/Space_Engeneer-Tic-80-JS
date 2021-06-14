# Space-Engineer-Tic-80-JS
Projeto pessoal: Jogo de nave misturado com Tetris


// title:  Space Engineer // author: Vinicius Noleto 
// desc:   Ship + Tetris  // script: js

var buttons={
 up:58,
 down:59,
 left:60,
 right:61,
 changeShipUp:23,
 changeShipDown:19,
 shotShipL:1,
 shotShipR:4,
 changeShipHeight:64
}

const constants={
 types:{
	 player:0,
	 blockS:1,
	 blockN:2,
		bullet:3,
		pixs:4,
	},
	screens:{
	 title:0,
		menu:1,
		survive:2,
		story:3,
		options:4,
		gameOver:5,
		
		tutorial1:8,
		tutorial2:9,
		tutorial3:10,
		tutorial4:11,
		tutorial5:12,
		tutorial6:13,
	}
}

var objects=[]
var background=[]
var screenHuds=[]
var soundEffects=[]

/////////////

var MASTER = function(_sprite,_height,
                      _x,_y,_tSize,_sizeX,
																						_sizeY,_velocity,_type){
 
	var self=this
	this.sprite=_sprite
	this.height=_height
	this.x=_x +(_sizeX*4)
	this.y=_y +(_sizeY*4)
	this.tSize=_tSize
	this.sizeX=_sizeX
	this.sizeY=_sizeY
	this.velocity=_velocity
	this.type=_type
	this.visible=true
	
	this.draw = function(self){
 if(self.visible){
			spr(self.sprite,
			    Math.round(self.x-(self.sizeX*4)), 
			    Math.round(self.y-(self.sizeY*4)),
       0,self.tSize,0,0,
       self.sizeX,self.sizeY)
  }
	}
	
}


/////////////////



var Celestial = function(_sprite,_height,
                         _x,_y,_tSize,_sizeX,
													            _sizeY,_velocity){
	
	MASTER.call(this,_sprite,_height,
             _x,_y,_tSize,_sizeX,
													_sizeY,_velocity)
													
	var self = this
	
	this.move = function(self){
	 self.x=self.x-self.velocity
	}
	
	this.att = function(){
	 self.draw(self)
		self.move(self)
	}								
 
	
}

function celestialBorn(_sprite,_height,
                       _x,_y,_tSize,_sizeX,
													          _sizeY,_velocity){
 
	var celestial = new Celestial(_sprite,_height,
                               _x,_y,_tSize,_sizeX,
													                  _sizeY,_velocity)
	
	background.push(celestial)
}

var starT=0
var planetT=0
var travelingPlanets=-1

function constellationEngine(){

	if(starT == 4){
	 var r = Math.round(Math.random()*(100-1)+1)
		var Y = Math.round(Math.random()*(180-1)+1)
		
		if(r<=30){
			celestialBorn(1,0,240,Y,1,1,1,0.6)
		}else if(r>30 && r<=60){
			celestialBorn(1,0,240,Y,1,1,1,0.8)
  }else if(r>60 && r<=90){
			celestialBorn(1,0,240,Y,1,1,1,0.9)
		}else if(r>90 && r<=100){
			celestialBorn(2,0,240,Y,1,1,1,4)
  }
		
		starT = 0
	}
	starT++
	
	
	if(planetT == 60*4){
	 
		var r = Math.round((Math.random()*(4-1))+1)
		var Y = Math.round((Math.random()*(131-5))+5)
		
		if(travelingPlanets==r && r!=4) r++
		if(travelingPlanets==r && r==4) r=1
		
		if(r==1){
			celestialBorn(16,0,240,Y,1,1,1,0.5)
		}else if(r==2){
			celestialBorn(17,0,240,Y,1,1,1,0.5)
  }else if(r==3){
			celestialBorn(18,0,240,Y,1,1,1,0.4)
  }else if(r==4){
			celestialBorn(19,0,240,Y,1,1,1,0.4)
  }
		
		travelingPlanets=r
		planetT=0
	 
	}
	planetT++

}


/////////////

var ObjectModel=function(_sprite,_height,
                         _x,_y,_ang,_tSize,
																									_sizeX,_sizeY,_velocity,
																									_type){
 
	MASTER.call(this,_sprite,_height,
             _x,_y,_tSize,_sizeX,
													_sizeY,_velocity,_type)
	
	var self = this
 
	this.ang=_ang
	this.freeze=false
	this.colli=[-(self.sizeX*4),(self.sizeX*4)-1,
	            -(self.sizeY*4),(self.sizeY*4)-1]
	
	var radAng = (self.ang * Math.PI)/180
	
	this.dx=self.velocity * Math.cos(radAng)
 this.dy=self.velocity * Math.sin(radAng)
	
	this.collision = function(self){
	 for(var i=0;i<objects.length;i++){ 
			if(objects[i].type != constants.types.pixes){
				if(self.x+self.colli[1]
				   >objects[i].x+objects[i].colli[0]
			 && self.x+self.colli[0]
				   <objects[i].x+objects[i].colli[1]
				&& self.y+self.colli[3]
				   >objects[i].y+objects[i].colli[2]
			 && self.y+self.colli[2]
				   <objects[i].y+objects[i].colli[3]
				&& self != objects[i]){
				 	self.hit(objects[i],self)
				}
		 }
		}
	}
	
	this.move = function(self){
	 if(!self.freeze){
		 self.dx=self.velocity * Math.cos(radAng)
		 self.dy=self.velocity * Math.sin(radAng)
		 
			self.x+=self.dx
	  self.y+=self.dy
		}
	} 
	
	this.actions = function(self){}
	
	this.att = function(){
		self.draw(self)
		self.collision(self)
		self.actions(self)
		self.move(self)
	}
	
	this.hit = function(obj,self){}
}

/////////////////////


var Player = function(){
 
	ObjectModel.call(
	 this,257,1,3*8,8*8,0,
		1,1,1,2,constants.types.player
	)
	
	var self = this
	
	this.pts = 0
	this.lifes = 5
	this.anger = 0
	this.reload = 0
	this.moveTimer = 0
 this.transform = 0
	
	this.nextSprite = self.sprite
	
	this.ammo = [
               [,,"missileLife",20],
	              [,,"freezeLife",4],
														 [,,"nitroLife",4],
														 [,,"rotateLife",15],
														 [,,"tractorLife",8]
														]
	
	this.missileLife = self.ammo[0][3]
	this.freezeLife = self.ammo[1][3]
	this.nitroLife = self.ammo[2][3]
	this.rotateLife = self.ammo[3][3]
	this.tractorLife = self.ammo[4][3]
	
	/////////////////////////////
	
	
	this.hit = function(obj,self){
  
		if((obj.type==1 && self.height==1)
		|| (obj.type==2 && self.height==2)){
		 self.visible=false
			
			screenHuds=[]
			hudBorn( 40,["Meu Deus, eles morreram...","...","Alo base? Vamos precisar de novos coletores..."])
		 
			pixBorn(360,self.x,self.y,2,self.height,90)
		}
		
 }
	
	this.move = function(self){
		
		self.dy=0
	 self.dx=0
	 
		if(key(buttons.up) && self.moveTimer==0){
			self.moveTimer = -4
		}
	 if(key(buttons.down) && self.moveTimer==0){
		 self.moveTimer = 4
		}
		
		if(self.moveTimer>0){
			self.moveTimer--
			self.dy+=(8/4)
		}else if(self.moveTimer<0){
		 self.moveTimer++
			self.dy+=-(8/4)
		}
		
	 if(key(buttons.left)) self.dx=-self.velocity
	 if(key(buttons.right)) self.dx=+self.velocity
		
		self.x+=self.dx
  self.y+=self.dy
	}
	
}

function playerMaker(){

	player = new Player()
	objects.push(player)
	
	player.indexForAmmoArrey = 
	 player.sprite-1*(257 +
		 (16*(player.height-1)))
	
	
	
	player.guideBow = function(self){
		if(self.visible){
			spr(263 + self.sprite-1*(257+(16*(self.height-1))),
		     self.x-7,self.y-4,0,1,0,0,1,1)
		}
		
	}
	
	player.lifeChange = function(_l){
	 player.lifes+=_l
		
		if(player.lifes>5) player.lifes=5
	}
	
	player.addPts = function(_pts,_b){
		_b = (typeof _b === 'undefined') ?
		 true : _b
		
		player.pts+=_pts
		if(_b){
		 sfx(13,'C-5',10,2,8)
		}
	}
	
	
	player.moodChange = function(_mood){
	 player.anger+=_mood
	}
	
	
	player.calmDown = function(self){
	 
		if(self.anger > 0){
		 self.angerTimer++
		}else{
		 self.angerTimer=0
		}
		
		if(self.angerTimer/60==1*60){
		 if(self.anger > 0) self.anger--
	 }
	}
	
	
	player.gun = function(self){
	 player.ammo[0][0]=Missile
		player.ammo[0][1]=Missile
		player.ammo[1][0]=Freeze
		player.ammo[1][1]=Freeze
		player.ammo[2][0]=Nitro
		player.ammo[2][1]=Nitro
		player.ammo[3][0]=RotateL
		player.ammo[3][1]=RotateR
		player.ammo[4][0]=Tractor
		player.ammo[4][1]=Tractor
				
		
		if(self.reload<20) self.reload++
		
		if(self.sprite != 256+(16*(self.height-1))
		&& self.reload == 20){
			
			self.indexForAmmoArrey = 
	   self.sprite-1*(257+(16*(self.height-1)))
	  
			if(self[self.ammo[self.indexForAmmoArrey][2]]>0
			&& key(buttons.shotShipL)){	
				bulletBorn(
				 self.ammo[self.indexForAmmoArrey][0],
				 self.x,self.y,self.height
				)
			}
			
			
			if(self[self.ammo[self.indexForAmmoArrey][2]]>0
			&& key(buttons.shotShipR)){			
				bulletBorn(
				 self.ammo[self.indexForAmmoArrey][1],
				 self.x,self.y,self.height
				)
			}
			
			if(key(buttons.shotShipL) || key(buttons.shotShipR)){
			 if(self[self.ammo[self.indexForAmmoArrey][2]]>0){
				 self[self.ammo[self.indexForAmmoArrey][2]]--
				}	
				
				sfx(10,'C-2',10,1,10)
				self.reload=0
			}
	 }
	}
	
	
	player.switchShip = function(self){
		if(self.transform<20) self.transform++
		
		
		if(self.transform>=14){
			
			if(key(buttons.changeShipUp)){
			 self.nextSprite--
					
				if(self.nextSprite==256+(16*(self.height-1))) self.nextSprite=261+(16*(self.height-1))
			}
			
			if(key(buttons.changeShipDown)){
			 self.nextSprite++
				
				if(self.nextSprite==262+(16*(self.height-1))) self.nextSprite=257+(16*(self.height-1))
			}
			
			if(key(buttons.changeShipUp) || key(buttons.changeShipDown)){
			 self.transform=0
				
				sfx(11,'G-3',10,1,10)
			}
			
		 for(var i = 0;i<5;i++){
		 	if(self[self.ammo[i][2]] < self.ammo[i][3]){
		 	 self[self.ammo[i][2]]+=2
		  }
	  }
			
		}
		
		if(key(buttons.changeShipHeight)
		&& self.transform ==20){
			
			if(self.nextSprite<272){
			
			 self.nextSprite+=16
				self.height=2
			 sfx(8,'E-4',10,1,10)
			
			}else{
			
			 self.nextSprite-=16
				self.height=1
			 sfx(9,'E-4',10,1,10)
			
			}
			
			self.transform=0
		}
		
		if(self.transform<=8){
		 self.sprite = 256+(16*(self.height-1))
		}else{
		 self.sprite = self.nextSprite
		}
		
	}
	
	player.actions = function(self){	
		self.gun(self)
	 self.switchShip(self)
		self.guideBow(self)
		
	 self.calmDown(self)	
	}

}

playerMaker()

/////////

var Bullet = function(_sprite,_color,_x,_y,_height){
 
	ObjectModel.call(
	 this,_sprite,_height,_x+2,_y-4,
		0,1,1,1,4,constants.types.bullet
	)
	
	this.colli = [-(3),(3)-1,
	              -(3),(3)-1]
	this.SBoom = function(){}
	this.NBoom = function(){}
	this.defaultBoom = function(){}
	
	this.actions = function(self){
	 if(self.x >=300) bulletsCount++ 
	}
	
	this.hit = function(obj,self){
		if((obj.type == constants.types.blockS && self.height==1)
		|| (obj.type == constants.types.blockN && self.height==2)){
		 for(var i=0;i<objects.length;i++){
	   if(objects[i].ID == obj.ID){
		   
					if(obj.type == constants.types.blockS){
					 self.SBoom(self,objects[i])
		   }else{
					 self.NBoom(self,objects[i])
		   }
					
					self.defaultBoom(self,objects[i])
		  }
	  }
			pixBorn(5,self.x,self.y,_color,self.height)
			
			self.visible=false
	 }
	}
}

var Missile = function(_x,_y,_height){
 Bullet.call(
		this,288,14,_x,_y,_height
	)
	
	this.defaultBoom = function(self,obj){
	 obj.visible=false
		sfx(12,'D-2',15,2,10)
	}
	
	this.NBoom = function(self,obj){	
	 player.addPts(1,false)
	 
	 pixBorn(50,obj.x,obj.y,14,obj.height,24)
	}
	
	this.SBoom = function(self,obj){
		if(obj.n==1){
		 hudBorn( 12,["","POR QUE VOCE EXPLODIU UM BLOCO VERDE SUA ANTA??"])
		 
			player.lifeChange(-2)
		 player.moodChange(3)
		}
			
		player.addPts(-5,false)
	 
		pixBorn(50,obj.x,obj.y,7,obj.height,24)
	}
}

var Freeze = function(_x,_y,_height){
 Bullet.call(
		this,289,11,_x,_y,_height
	)
	
	this.defaultBoom = function(self,obj){
	 obj.freeze=true
		obj.freezeTimer=60*3
		obj.sprite=34
	}
}

var Nitro = function(_x,_y,_height){
 Bullet.call(
		this,290,8,_x,_y,_height
	)
	
	this.defaultBoom = function(self,obj){
		obj.nitroTimer=60*2
	}
	
	this.NBoom = function(self,obj){
	 obj.velocity = 1.4
	}
	
	this.SBoom = function(self,obj){
	 obj.velocity = 1
	}
}

function rotateBlock(obj,_r,_x,_y){
	
	var FORMAT
	
	const rotateArrey = [
	 [[[0,0],[0,8],[0,16],[0,24]],
		 [[0,0],[8,0],[16,0],[24,0]],
			[[0,0],[0,-8],[0,-16],[0,-24]],
			[[0,0],[-8,0],[-16,0],[-24,0]]],
		
		[[[0,0],[8,0],[8,8],[16,8]],
		 [[0,0],[0,-8],[8,-8],[8,-16]],
			[[0,0],[-8,0],[-8,-8],[-16,-8]],
			[[0,0],[0,8],[-8,8],[-8,16]]],
		
		[[[0,0],[8,0],[16,0],[16,8]],
		 [[0,0],[0,-8],[0,-16],[8,-16]],
			[[0,0],[-8,0],[-16,0],[-16,-8]],
			[[0,0],[0,8],[0,16],[-8,16]]],
		
		[[[0,0],[0,8],[8,0],[8,8]],
		 [[0,0],[0,8],[8,0],[8,8]],
			[[0,0],[0,8],[8,0],[8,8]],
			[[0,0],[0,8],[8,0],[8,8]]],
		
		[[[0,0],[8,0],[8,-8],[16,0]],
		 [[0,0],[0,-8],[-8,-8],[0,-16]],
			[[0,0],[-8,0],[-8,8],[-16,0]],
			[[0,0],[0,8],[8,8],[0,16]]]
	]
	
	if(obj.format == "I"){
	 FORMAT=0 
	}else if(obj.format == "R"){
	 FORMAT=1
	}else if(obj.format == "L"){
	 FORMAT=2
	}else if(obj.format == "S"){
	 FORMAT=3
		if(obj.n==1){
			hudBorn(8,["E serio que voce tentou girar...", "Um bloco quadrado..?"])
		 if(player.anger<2){
			 hudBorn( 44,["Eu so esbarrei no botao, chefia","Sou tao burro assim nao"])
	  }else if(player.anger>=2 && player.anger<4){ 
			 hudBorn( 44,["Nao amigao, acabou de entrar um fantasma","aqui e ele atiorou, olha so"])
	  }else if(player.anger>=5){
			 hudBorn( 76,["TEMOS UM DETETIVE AQUI, OLHA SO!","UM GENIO, EU DIRIA!!"])
			}
		 player.moodChange(1)
		}
	}else if(obj.format == "C"){
	 FORMAT=4
	}
	
	obj.r=_r
	
	if(FORMAT==3)obj.r=0
	 
	if(obj.r==4) obj.r=0
	if(obj.r==-1) obj.r=3
		
	obj.x=_x+rotateArrey[FORMAT][obj.r][obj.n-1][0]
	obj.y=_y+rotateArrey[FORMAT][obj.r][obj.n-1][1]
	
}

var RotateL = function(_x,_y,_height){
 Bullet.call(
		this,291,6,_x,_y,_height
	)
	
	var X,Y,R
	var rotated = false
	
	this.defaultBoom = function(self,obj){
	 
		if(!obj.grabbed){
		 if(obj.n==1 && !rotated){
				obj.r++
				R=obj.r
				X=obj.x
				Y=obj.y
				rotated=true
			}
			
			if(typeof R != 'undefined'){
			 rotateBlock(obj,R,X,Y)
			}
	 }
	}
}

var RotateR = function(_x,_y,_height){
 Bullet.call(
		this,307,6,_x,_y,_height
	)
	
 var X,Y,R
	var rotated = false
	
	this.defaultBoom = function(self,obj){
	 if(!obj.grabbed){
			if(obj.n==1 && !rotated){
				obj.r--
				R=obj.r
				X=obj.x
				Y=obj.y
				rotated=true
			}
		}
		if(typeof R != 'undefined'){
		 rotateBlock(obj,R,X,Y)
		}
	}
}

var Tractor = function(_x,_y,_height){
 Bullet.call(
		this,292,12,_x,_y,_height
	)
	
	var self = this
	
	this.draw = function(self){
	 line(player.x+5,Math.round(player.y)-4,self.x,self.y,13)
	 line(player.x+5,Math.round(player.y)+3,self.x,self.y,13)
	}
	
	this.defaultBoom = function(self,obj){
	 if(!obj.grabbed){
			obj.freeze=true
			obj.grabbed=true
			
			obj.grabbDY=player.y-obj.y
			obj.freezeTimer=12
			obj.grabbTimer=60*4
			
	 }else{
			obj.grabbed=false
		}
	}
	
	this.NBoom = function(self,obj){
	 obj.sprite=49
	}
	
	this.SBoom = function(self,obj){
	 obj.sprite=48
	}
}

function bulletBorn(TYPE,_x,_y,_height){
 
	var bullet = new TYPE(_x,_y,_height)
										
 objects.push(bullet)
}

////////////////////////////////////////

var Block = function(_x,_y,_type,_ID,_n,_format){
 
	var v = [0.4,0.8]
	
	ObjectModel.call(
	 this,31+_type,_type,_x+240,_y,
		180,1,1,1,v[_type-1],_type
	)
	
	var self = this
	
	this.n = _n
	this.ID = _ID
	this.format = _format
	
	this.r=0
	this.freezeTimer=0
	this.nitroTimer=0
	this.grabbTimer=0
	this.grabbDY=0
	this.grabbed = false
	this.haveWay = false
	this.pointEarned = false
	
	var radAng = (self.ang * Math.PI)/180
	
	this.move = function(self){
	 if(!self.freeze){
		 self.dx=self.velocity * Math.cos(radAng)
		 self.dy=self.velocity * Math.sin(radAng)
		 
			self.x+=self.dx
		 self.y+=self.dy
		}
	}
	
	this.freezeAction = function(self){
	 if(self.freezeTimer>0){
		 self.freezeTimer--
		}
		
		if(self.freezeTimer==1){
			var oldSprite = [32,33]
			
			self.sprite=oldSprite[self.type-1]
			
			self.freeze=false
		}
	}
	
	this.nitroAction = function(self){
	 if(self.nitroTimer>0){
		 self.nitroTimer--
		}
		
		if(self.nitroTimer==1){
		 var oldV = [0.4,0.8]
			
			self.velocity = oldV[self.type-1]
		}
	}
	
	this.grabbAction = function(self){
	 if(self.grabbed){
		 if(self.y < player.y-self.grabbDY){
				self.y++
			}else if(self.y > player.y-self.grabbDY){
			 self.y--
			}
			
			if(self.grabbTimer==0){
			 self.grabbed=false
			}
			
			self.grabbTimer--
			self.freezeTimer=12
		}
	}
	
	this.actions = function(self){
		
		self.freezeAction(self)
		self.nitroAction(self)
		self.grabbAction(self)
		
	}
	
	this.hit = function(obj,self){
	 if(obj.height == self.height
		&&(obj.type == constants.types.blockN
		|| obj.type == constants.types.blockS)
		&& obj.ID != self.ID){
			for(var i=1;i<objects.length;i++){
			 if(objects[i].ID == self.ID
				|| objects[i].ID == obj.ID){
					var c = [7,14]
					
			  pixBorn(50,objects[i].x,objects[i].y,
					        c[objects[i].type-1],objects[i].height,24)
		  
				 objects[i].visible=false
					
					sfx(12,'D-2',15,2,10)
				}
			}
		}
	}
	
	
}

function blockBorn(_x,_y,_type,_ID,_n,_format){

	block = new Block(_x,_y,_type,_ID,_n,_format)
	
	objects.push(block)
}


///////////////////////

var totalBlockID = 0
var blockT = 0
var blockSTimer=0
var timeLess = 0

function blobckBornEngine(){	
	
	if(actualScreen == constants.screens.survive){
		timeLess = Math.floor(Math.floor(gameTotalTime/60)*0.1)
	}else if(actualScreen == constants.screens.story){
	 timeLess=(level-1)*20
	}
	
	if(blockT>=60*2 - timeLess){
	 
	 totalBlockID++
	
	 var Y = Math.round((Math.random()*(14-1))+1)
  var r = Math.round((Math.random()*(5-1))+1)
	 var cR = Math.round(Math.random()*100)
		var B1,B2,B3,B4,_type,F
		
	 if(r==1){
	  B1 = [0,-1]
		 B2 = [0, 0]
		 B3 = [0, 1]
		 B4 = [0, 2]
			F = "I"
	 }else if(r==2){
	  B1 = [0,0]
		 B2 = [1,0]
		 B3 = [1,1]
		 B4 = [2,1]
			F = "R"
	 }else if(r==3){
	  B1 = [0,0]
		 B2 = [1,0]
		 B3 = [2,0]
		 B4 = [2,1]
			F = "L"
	 }else if(r==4){
	  B1 = [0,0]
		 B2 = [0,1]
		 B3 = [1,0]
		 B4 = [1,1]
			F = "S"
	 }else if(r==5){
	  B1 = [0,1]
		 B2 = [1,1]
		 B3 = [2,1]
		 B4 = [1,0]
			F = "C"
	 }
		
		if(blockSTimer>0){
		 _type=constants.types.blockN
			blockSTimer--
		}else{
		 _type=constants.types.blockS
			blockSTimer = Math.round((Math.random()*(8-4))+4)
		}
		
	 blockBorn(8*B1[0],(8*B1[1])+(Y*8),_type,totalBlockID,1,F)
  blockBorn(8*B2[0],(8*B2[1])+(Y*8),_type,totalBlockID,2,F)
  blockBorn(8*B3[0],(8*B3[1])+(Y*8),_type,totalBlockID,3,F)
  blockBorn(8*B4[0],(8*B4[1])+(Y*8),_type,totalBlockID,4,F)
  
	 blockT=0
	}else{
	 blockT++
	}
}


////////////////////////////////////////////////

var Pix = function(_x,_y,_life,_color,_height){
	
	MASTER.call(this,0,_height,
             _x,_y,1,1,1,1,
													constants.types.pixes)
	
	
	var self = this
			
	this.life=_life
	this.color=_color
	this.height=_height
	this.ang=Math.round(Math.random()*360)
	this.dx=Math.cos(self.ang)
	this.dy=Math.sin(self.ang)
	
	this.move = function(self){
	 self.x+=self.dx*2
		self.y+=self.dy*2
	 
		self.life--
	}
	
	this.die = function(self){
	 if(self.life<=0){
		 self.visible=false
		}
	}
	
	this.draw = function(self){
	 if(self.visible){
		 pix(self.x,self.y,self.color)
		}
	}
	
	this.att = function(){
	 self.die(self)
		self.draw(self)
		self.move(self)
	}
	
}

function ramdomLife(){
 return Math.round(Math.random()*12) - (Math.round(Math.random()*12))
}

function pixBorn(N,_x,_y,_color,_height,_life){
 _life = (typeof _life === 'undefined') ?
	  12 : _life
	
	
	for(var i=1;i<=N;i++){
	 
		var PIX = new Pix(_x,_y,_life+ramdomLife(),_color,_height)
	 	
		objects.push(PIX)
	}
}

///////////////////////////////////////

var hud = function(_sprite,_txt){
 MASTER.call(this,396,0,1*8,14*8,1,2,2)
	
	var self = this
	
	this.t = 0
	this.ani = 0
	this.actualSprite=0
	
	this.spriteArrey = [396,   398,   428,
	                    430,   460,   462,
																					490,   492,
	                    _sprite,_sprite+2,
																					492,   494,   426,   
																					428,   398,   396]
	
	this.spriteAtt = function(self){
	 self.ani++
		
		if((self.actualSprite < 8 
		|| self.actualSprite >= 10)
		&& self.ani==6){
		 
			self.actualSprite++
	  self.ani=0
		}
		
		if(self.actualSprite >= 8
		&& self.actualSprite < 10){
		 
			for(var i=0;i<_txt.length;i++){
		 	print(_txt[i],4*8,(14+i)*8,1,false,1,true)
			}
			
			if(self.ani==8){
				self.actualSprite++
				self.t++
			 sfx(14,'A#4',15,3,2)
				
				self.ani=0
				
				if(self.t/8 < 4
				&& self.actualSprite==10){ 
				 self.actualSprite = 8
			 }
			}
		}
		
		if(self.actualSprite > 15){
		 self.visible = false
		}
		
		self.sprite = self.spriteArrey[self.actualSprite]
	}
	
	this.att = function(){
		self.draw(self)
		self.spriteAtt(self)
	}
	
}

function hudBorn(_sprite,_txt){
 
	var HUD = new hud(_sprite,_txt)
	
	screenHuds.push(HUD)
}

var justOne = 0

function randomTalks(){
	if(!timeInSec(40)) justOne = 0
	
 if(timeInSec(40) && justOne==0){
		justOne++
		
		var r = Math.round(Math.random()*(4-1))+1
		
		if(r==1){		
			
			hudBorn(136,["Se liga no oculos novo que eu comprei"])
			hudBorn( 72,["Ah","Legal, cara, parabens"])
			hudBorn(168,["Voce nao gostou? :("])
			hudBorn( 44,["Claro que ***!"])
		}else if(r==2){
			
			hudBorn(172,["*chorando de solucar*"])
			hudBorn( 44,["Qual foi, man?"])
			hudBorn(172,["Derramei meu cafe no chao","Nunca fiquei tao triste na vida :(("])
			hudBorn( 44,["Eu te pago um depois, cara","So se concetra ai pfvr"])
		}else if(r==3){
		 
			hudBorn(200,["Ei, fiz um pao caseiro pra gente :)","Ta quentinho ainda!"])
		 hudBorn( 72,["","Calma la, como assim tem um forno na nave??"])
		 hudBorn( 76,["E POR QUE VOCE TA COZINHANDO","EM UM MOMENTO DESSE???"])
		 hudBorn(168,["","Poxa mas eu fiz ate um Queijinho Minas... :("])	
		 
			player.moodChange(1)
		}else if(r==4){
		 
			hudBorn(204,["Voce tem quantos anos mesmo?"])
		 hudBorn( 72,["Tenha 28, por que?"])
			hudBorn(204,["Nesse livro aqui diz que metade dos","pilotos morrem no intervalo de 23 a 32 anos","em acidentes do espaco"])
		 if(player.anger<=3){
			 hudBorn( 44,["Uau","Estou me sentindo suuuper seguro agora, ein..."])
			}else{
			 hudBorn( 76,["Sabe oq e bom pra nao ocorrer um acidente...?","NAO LER DURANTE O TRABALHO >:("])
			}
		}
	}
}	

var bulletsCount = 0

function talkEvents(){
 if(bulletsCount==5){
	 
		 hudBorn( 12,["Nao gaste municoes assim,","o Imperio paga caro por elas!"])
  if(player.anger<5){
		 hudBorn( 44,["OK...","Vou tentar melhorar"])
	 }else if(player.anger>=5){
		 hudBorn( 76,["Vem aqui entao oh bonzao","Pilota e atira pra ver se e facil"])
  }
		
		bulletsCount=0
		player.moodChange(2)
	
	}
 
}

function scoreboard(){
 spr(296,13.4*8,0,0,1,0,0,3,1)
	print(Math.round(player.pts),1+(13.4*8),2,1)
}

////////

var wayToBlock = []

var flasherLight = 0

function wayToGoEngine(){
 var IDToWay = -1
	var blockFormat
 
	for(var i=0;i<objects.length;i++){
	 if(objects[i].sprite==32
		&& !objects[i].haveWay){
		 IDToWay = objects[i].ID
			blockFormat = objects[i].format
			objects[i].haveWay = true
		}
	}
	
	if(IDToWay != -1){
	 var Y = Math.round(Math.random()*15)
	 var r = Math.ceil((Math.random()*(4-1))+1)
		var S
		var ROptions = [,,,]
		var Limt = [,,,]
		
		if(blockFormat=="I"){
		 S=331+Math.floor(r/2)-1
			Limit = [4,4,1,1]
			ROptions = [[0,2],[0,2],[1,3],[1,3]]
		}else if(blockFormat=="R"){
		 S=328+Math.floor(r/2)-1
		 Limit = [2,2,3,3]
			ROptions = [[0,2],[0,2],[1,3],[1,3]]
		}else if(blockFormat=="L"){
		 S=320+r-1
		 Limit = [2,3,2,3]
			ROptions = [[0,0],[1,1],[2,2],[3,3]]
		}else if(blockFormat=="S"){
		 S=330
		 Limit = [2,2,2,2]
			ROptions = [[0,0],[0,0],[0,0],[0,0]]
		}else if(blockFormat=="C"){
		 S=324+r-1
		 Limit = [2,3,2,3]
			ROptions = [[0,0],[1,1],[2,2],[3,3]]
		}
	 
		if(Y>13 && blockFormat=="I"){
		 Y=13
		}
		if((blockFormat=="R" || blockFormat=="S"
		|| blockFormat=="C") && Y>14){
		 Y=14
		}
		
		wayToBlock.push([S,Y*8,IDToWay,true,Limit[r-1]*8,ROptions[r-1]])
	}
	
	flasherLight++
	
	for(var i=0;i<wayToBlock.length;i++){
	 if(wayToBlock[i][3]
		&& !(flasherLight%60<20)){
		 spr(wayToBlock[i][0],0,
		     wayToBlock[i][1],0,1,0,0,1,4)
	 }
	}
	
	for(var i=0;i<wayToBlock.length;i++){
	 var destroy = true
	 for(var j=0;j<objects.length;j++){
	  if(wayToBlock[i][2]==objects[j].ID){
		  destroy = false
		  
				if(objects[j].y >= wayToBlock[i][1]
				&& objects[j].y <= wayToBlock[i][1]+wayToBlock[i][4]
				&& !(objects[j].pointEarned)
				&& objects[j].x <= 0){
				 
					if(objects[j].r == wayToBlock[i][5][0]
					|| objects[j].r == wayToBlock[i][5][1]){
					 player.addPts(10)
						if(player.lifes<5)player.lifeChange(0.1)
						blockSTimer--
					}else{
					 player.addPts(5)
					}
					
					objects[j].pointEarned=true
					
				}else if((objects[j].y < wayToBlock[i][1]
				|| objects[j].y > wayToBlock[i][1]+wayToBlock[i][4])
				&& !(objects[j].pointEarned)
				&& objects[j].x <= 0){
				 player.addPts(-2.5)
					player.lifeChange(-0.125)
					
					objects[j].pointEarned=true
				}
				
			}
	 }
	 if(destroy){
	  wayToBlock[i][3]= false
		}
	}
	
}

//////////////////////////

var Boss = function(_txt){
 ObjectModel.call(
	 this,64,3,30*8,8*8,180,
		1,3,3,0.8,5
	)
	
	var self = this
	
	this.txt = function(){
	 for(var i=0;i<_txt.length;i++){
		 print(_txt[i],4*8,(14+i)*8,1,false,1,true)
		}
	}
	
	this.actions = function(){
	 if(self.x<=24*8) self.velocity=0
		
		if(self.finishedTalk){
		 self.velocity = 0.8
			self.ang = 0
		}
	}
}

var fired = false
var firedT = 0
var END = false

function playerLose(){
	
	if(player.lifes <=0){
	 if(!fired){
		 screenHuds=[]
		 hudBorn( 12,["Voce e um pessimo coletor esta DEMITIDO!!"])
		 firedT++
		}
		
		fired=true
	}
	
	///
	
	var playerInGame = false
	for(var i = 0;i<objects.length;i++){
	 if(objects[i].type == constants.types.player) playerInGame=true 
	}
	
	if(!playerInGame){
		
		for(var i = 0;i<objects.length;i++){
	  if(objects[i].type == constants.types.player) objects.splice(i,1)
	 }
		if(!END){
		 pixBorn(360,player.x,player.y,2,player.height,90)
	  gameTotalTime=0
		}
		
		END = true
	}
	
	//////////
	
	if(firedT >=1 && firedT <60){
	 firedT++
	}else if(firedT == 60){
	 firedT++
		END = true
		gameTotalTime=0
	}
	
	if(END){
	 if(timeInSec(5)){
			fired = false
			firedT = 0
			END = false
			
			bossTime = false
			finalDialogue = true
			level = 1
			
			bulletsCount = 0
			blockSTimer=0
			
			objects=[]
			playerMaker()
			
			if(actualScreen == constants.screens.story
			|| actualScreen == constants.screens.survive){
			 actualScreen = constants.screens.gameOver+actualScreen-2
		 }else{
			 actualScreen = constants.screens.gameOver+2
			}
		}
	}
 
}


///////////

var bossTime = false
var finalDialogue = true
var level = 1
var foundationX = 240+24
var bossSprite = 0
var creditsY = 0

function defaultGameEngine(){
 if(actualScreen == constants.screens.survive
	|| actualScreen == constants.screens.story){
	 wayToGoEngine()
	 blobckBornEngine()
	 playerLose()
	}
}

function congratulations(){
 
	if(finalDialogue){
		gameTotalTime=0
	}
	finalDialogue=false
 
	
	bossSprite+=0.1
	if(bossSprite>=2) bossSprite=0
	
	foundationX-=0.5
	
	
	spr(64+((level-1)*48)+Math.floor(bossSprite)*3,
	    foundationX,7*8,0,1,0,0,3,3)
	spr(208+((level-1)*16),
	    foundationX-18,8*8,0,1,0,0,2,1)
	
	print("Parabens coletores!!",
	      foundationX,5*8,1,false,1,true)
	print("Voces contruibuiram com o Imperio!",
	      foundationX,6*8,1,false,1,true)
	
	print("aperte 'z' para pular",10.5*8,14*8,2,false,1,true)
	
	if(btn(4)){
	 pixBorn(45,foundationX-18,8*8,4,0,14)
	 pixBorn(45,foundationX-18,8*8,1,0,16)
	 
		var c = [11,15,13]
	 pixBorn(90,foundationX,8*8,2,0,28)
	 pixBorn(90,foundationX,8*8,c[level-1],0,32)
	}
	if(timeInSec(14) || btn(4)){
	 foundationX = 240+24
		bossTime=false
		finalDialogue = true
		level++
	}
}

function credits(){
 if(finalDialogue){
	 hudBorn(  8,["Obrigado, sem voce eu nunca seria promovido :)","...","Mas isso nao quer dizer que seu salario vai aumentar."])
	}
	finalDialogue = false
	
	print("Em breve mais atualizacoes...",0,creditsY-48,1)
	print("GitHub: viniciusNoleto",0,creditsY-24,1)
	print("Desenvolvedor: Vinicius Noleto de Araujo",0,creditsY-16,1)
 print("Muito obrigado por jogar :)",0,creditsY,1)
		 
	creditsY+=0.1
}

function gameEngine(){
 if(timeInSec(60*3) && actualScreen == constants.screens.story) bossTime=true
	
	if(!bossTime){
	 
		defaultGameEngine()
	
	}else{
		if(level<=3){
		 congratulations()
		}else{
		 credits()
		}
	}
}

////////


function hudEngine(){
 if(!bossTime){
		randomTalks()
		talkEvents()
 }
}

////////


function objAtt(array){
 for(var H=0;H<=3;H++){
	 for(var i=0;i<array.length;i++){
		 if(array[i].height==H){
			
			 if((array[i].type == constants.types.player
				&& actualScreen != constants.screens.gameOver
				&& actualScreen != constants.screens.gameOver+1
				&& actualScreen != constants.screens.gameOver+2)
				|| array[i].type != constants.types.player){
				 
					array[i].att()
				
				}
			
			}
	 }
	}
}

function hudAtt(){
 if(screenHuds.length>0){
		screenHuds[0].att()
 }
}

function objEliminate(array,cond1,cond2){
 for(var i=0;i<array.length;i++){
	 if(array[i].x<=cond1 - (array[i].sizeX*4)
		|| array[i].x>=cond2 + (array[i].sizeX*4)
		|| !array[i].visible){
		 array.splice(i,1)
		}
	}
}

function notVisibleEliminate(array){
 for(var i=0;i<array.length;i++){
	 if(array[i].mute === true 
		|| array[i].visible === false){
	  array.splice(i,1)
	 }
	}
}

function worldBackground(){
 
	cls()
	
	objAtt(background)
	
	constellationEngine()
	
	objEliminate(background,-24,248)
}

function justExplosionsAtt(){
 for(var i=0;i<objects.length;i++){
		if(objects[i].type==constants.types.pixes){
		 objects[i].att()
		}
	}
}

function worldObjects(){
 gameEngine()
	
	objAtt(objects)
	
	objEliminate(objects,-24,300)
}

function worldSoundAndHud(){
 scoreboard()
	
	if(actualScreen == constants.screens.survive
	|| actualScreen == constants.screens.story){
	 hudEngine()
	}
	
	hudAtt()
	
	objAtt(soundEffects)
	
	notVisibleEliminate(soundEffects)
	
	notVisibleEliminate(screenHuds)
}

////////////////////

KEYCODES = [

"A","B","C","D","E","F","G","H",
"I","J","K","L","M","N","O","P",
"Q","R","S","T","U","V","W","X",
"Y","Z",

"0","1","2","3","4","5","6","7",
"8","9",

"MINUS","EQUALS","LEFTBRACKET",
"RIGHTBRACKET","BACKSLASH",
"SEMICOLON","APOSTROPHE","GRAVE",
"COMMA","PERIOD","SLASH","SPACE",
"TAB","RETURN","BACKSPACE",
"DELETE","INSERT","PAGEUP","PAGEDOWN",
"HOME","END",

"UP","DOWN","LEFT","RIGHT",

"CAPSLOCK","CTRL","SHIFT","ALT",

]

var changeOptionsArrey = [
	"changeShipUp","changeShipDown",
	"shotShipR","shotShipL",
	"changeShipHeight",
	"up","down","left","right"
]

function buttonsChange(){
	
	if(btn(0) && arrowSelector.velocity==0){
	 buttons[changeOptionsArrey[arrowSelector.cord]]++
		
		if(buttons[changeOptionsArrey[arrowSelector.cord]]>KEYCODES.length){
		 buttons[changeOptionsArrey[arrowSelector.cord]]=1
		}
		
		arrowSelector.velocity=6
	}
	
	if(btn(1) && arrowSelector.velocity==0){
	 buttons[changeOptionsArrey[arrowSelector.cord]]--
		
		if(buttons[changeOptionsArrey[arrowSelector.cord]]<=0){
		 buttons[changeOptionsArrey[arrowSelector.cord]]=KEYCODES.length
		}
		
		arrowSelector.velocity=6
	}
	
}

function printc(_s,_x,_y,_c,_m,_f){
 var w=print(_s,0,-12)
 print(_s,_x-(w/2),_y,_c,false,_m,_f)
}

function showActualButtons(){

 printc("Nave Anterior",5*8,3*8,1,1,true)
	printc("Proxima Nave",12.8*8,3*8,1,1,true)
	printc("Tiro Primario",20*8,3*8,1,1,true)
	printc("Tiro Secundario",28*8,3*8,1,1,true)
	
	printc("Altura da Nave",134,7*8,1,1,true)
	
	printc("Mover-se",4.2*8,11*8,1,1,true)
	printc("(cima)",3.6*8,12*8,1,1,true)
	printc("Mover-se",12*8,11*8,1,1,true)
	printc("(baixo)",11.8*8,12*8,1,1,true)
	printc("Mover-se",19.2*8,11*8,1,1,true)
	printc("(esquerda)",19.4*8,12*8,1,1,true)
	printc("Mover-se",27.2*8,11*8,1,1,true)
	printc("(direita)",27.1*8,12*8,1,1,true)
	
	
	printc(KEYCODES[buttons.changeShipUp-1],3.2*8,2.2*8,6,1,false)
	printc(KEYCODES[buttons.changeShipDown-1],11.2*8,2.2*8,6,1,false)
	printc(KEYCODES[buttons.shotShipR-1],18.5*8,2.2*8,6,1,false)
	printc(KEYCODES[buttons.shotShipL-1],26.4*8,2.2*8,6,1,false)
	
	printc(KEYCODES[buttons.changeShipHeight-1],120,6*8,6,1,false)
	
	printc(KEYCODES[buttons.up-1],3.2*8,10.2*8,6,1,false)
	printc(KEYCODES[buttons.down-1],11.2*8,10.2*8,6,1,false)
	printc(KEYCODES[buttons.left-1],18.4*8,10.2*8,6,1,false)
	printc(KEYCODES[buttons.right-1],26.3*8,10.2*8,6,1,false)
	
	printc("Use as setas para navegar e mudar os valores",165,14.8*8,6,1,true)
	printc("Aperte 'z' ou 'x' para sair e confirmar",154,16*8,2,1,true)
}

function options(){
 showActualButtons()
	buttonsChange()
}

////////////////////

var titleShipY = 5*8
var titleShipV = 0.1
	
function gameTitle(){
	
	if(actualScreen == constants.screens.title){
	 spr(468,9*8,3*8,0,2,0,0,6,3)
		
		titleShipY = 5*8
		titleShipV = 0.1
 }else if(actualScreen == constants.screens.menu){
	 if(titleShipY==5*8){
			pixBorn(30,14.4*8,5.5*8,2,1,18)
		}
		
		spr(420,9*8,3*8,0,2,0,0,6,3)
		
		if(titleShipY > 790){
		 spr(388,13*8,5*8,0,2,0,0,2,1)
		}else if(titleShipY <= 790 && titleShipY > 780){	
		 spr(390,13*8,5*8,0,2,0,0,2,1)
		}else if(titleShipY <= 780 && titleShipY > 770){	
		 spr(392,13*8,5*8,0,2,0,0,2,1)
		}else if(titleShipY <= 770 && titleShipY > 760){	
		 spr(404,13*8,5*8,0,2,0,0,2,1)
		}else if(titleShipY <= 760 && titleShipY > 750){	
		 spr(406,13*8,5*8,0,2,0,0,2,1)
		}else if(titleShipY <= 750 && titleShipY > 5*8){	
		 spr(408,13*8,5*8,0,2,0,0,2,1)
		}
		
		spr(497,13*8,titleShipY,0,2,0,0,2,1)
		
		if(titleShipV<2)titleShipV+=0.01
		
		titleShipY-=titleShipV
  
		if(titleShipY<-200) titleShipY=800
	}
}

var tutorialRead=false

function gameInfo(){
 if(actualScreen == constants.screens.title){
	 print("aperte 'z' para continuar",9.1*8,12*8,2,false,1,true)
	}else if(actualScreen == constants.screens.menu){
	 
		spr(416,5*8,11*8,0,1,0,0,4,1)
  
		spr(432,10.5*8,11*8,0,1,0,0,4,1)
  
		spr(448,15.5*8,11*8,0,1,0,0,4,1)
  
		spr(464,20*8,11*8,0,1,0,0,4,1)
  
		tutorialRead=false
		
	}else if(actualScreen == constants.screens.tutorial1){
	 print("aperte 'z' para pular o dialogo",7.8*8,11*8,2,false,1,true)
		
		print("Setas:",7.8*8,1.5*8,15)
		print("Movimenta a nave",8*8,2.5*8,1,false,1,true)
		
		print("W e S:",7.8*8,3.5*8,15)
		print("Muda a nave",8*8,4.5*8,1,false,1,true)
		
		print("A e D:",7.8*8,5.5*8,15)
		print("Atira (o tiro da nave verde",8*8,6.5*8,1,false,1,true)
		print("varia com o botao apertado)",8*8,7.5*8,1,false,1,true)
		
		print("Shift:",7.8*8,8.5*8,15)
  print("Muda a altura do voo da nave",8*8,9.5*8,1,false,1,true)
		
		if(!tutorialRead){
		
			hudBorn(  8,["Bom, sua missao ja vai comecar e ela e simples:","Colete os blocos verdes e destrua","o vermelhos se possivel"])
	  hudBorn(  8,["Os blocos verdes possuem minerios que serao usados","na base do Imperio que esta em construcao","Entao tenha cuidado com eles!"])
	  hudBorn( 44,["Beleza, algo mais?"])
	  hudBorn(  8,["Voce tera um engenheiro que vai na sua nave com voce","Qualquer porblema ele e o encarregado"])
	  hudBorn(104,["Opa, tudo certo?"])
	  hudBorn( 72,["Perai por que so voce tem nariz??","Por que eu fui desenhado sem???","Programador preguicoso..."])
	  hudBorn(  8,["Isso nao importa","Facam o trabalho de voces pra eu","poder ser promovido um dia.. Um viva ao imperio!!"])
			hudBorn( 44,["Isso ai... Imperio... ebaaa..."])
	  hudBorn(104,["Uma dica antes da missao, lembre-se que comecamos","na altura dos blocos verdes","Se quiser acertar os vermelhos mude a altura"])
	  
			tutorialRead=true
		}
	}else if(actualScreen == constants.screens.tutorial2){
	 print("aperte 'z' para pular o dialogo",7.8*8,11.5*8,2,false,1,true)
		
		print("Nave Vermelha:",7.8*8,1.2*8,15)
		print("Missil, destroi os blocos",8*8,2.2*8,1,false,1,true)
		
		print("Nave Azul:",7.8*8,3.2*8,15)
		print("Tiro Gelado, congela os blocos",8*8,4.2*8,1,false,1,true)
		
		print("Nave Amarela:",7.8*8,5.2*8,15)
		print("Nitro, acelera os blocos",8*8,6.2*8,1,false,1,true)
		
		print("Nave Verde:",7.8*8,7.2*8,15)
  print("Tiro Rotacionador, gira os blocos",8*8,8.2*8,1,false,1,true)

		print("Nave Roxa::",7.8*8,9.2*8,15)
  print("Raio Trator, move os blocos",8*8,10.2*8,1,false,1,true)
		
		if(tutorialRead){
			hudBorn(236,["Ola, eu serei seu instrutor de tiro e pilotagem","Tentarei ser o mais claro possivel"])
		 hudBorn(236,["Os blocos verdes vem da direita e seus locais","de coleta estao indicados na esquerda","Assim como suas posicoes"])
		 hudBorn(236,["Use a nave verde para girar os blocos e","a roxa para move-las para o local correto"])
		 hudBorn( 44,["Ok, parece facil"])
		 hudBorn(204,["Apenas lembre-se de ter cuidado com a nossa nave,","ela e extremamente fragil"])
	  hudBorn(  8,["Seu foco sao os blocos verdes, mas se possivel","destrua os vermelhos, assim eu sou promovido mais","rapido"])
	  
			tutorialRead=false
		}
	}else if(actualScreen == constants.screens.tutorial3){
	 if(!tutorialRead){
		 hudBorn(236,["Primeiro: Atire com a nave roxa no bloco verde","Apos isso leve-a para o local indicado"])
		}
		tutorialRead=true
		
		if(timeInSec(10)) hudBorn(236,["Primeiro: Atire com a nave roxa no bloco verde","Apos isso leve-a para o local indicado"])
	
	}else if(actualScreen == constants.screens.tutorial4){
	 if(tutorialRead){
		 hudBorn(236,["Segundo: Atire com a nave verde no bloco verde","Isso vai girar o bloco para que ele fique","na posicao indicada"])
		}
		tutorialRead=false
		
		if(timeInSec(10)) hudBorn(236,["Segundo: Atire com a nave verde no bloco verde","Isso vai girar o bloco para que ele fique","na posicao indicada"])
		
	}else if(actualScreen == constants.screens.tutorial5){
	 if(!tutorialRead){
		 hudBorn(236,["Agora gire o bloco ate a posicao correta e, apos isso,","coloque-o no local indicado"])
		}
		tutorialRead=true
		
		if(timeInSec(10)) hudBorn(236,["Agora gire o bloco ate a posicao correta e, apos isso,","coloque-o no local indicado"])
		
	}else if(actualScreen == constants.screens.tutorial6){
	 if(tutorialRead){
		 hudBorn(236,["Por ultimo mude a altura da nave apertando Shift,","mude para a nave vermelha e destrua os","blocos vermelhos"])
		}
		tutorialRead=false
		
		if(timeInSec(10)) hudBorn(236,["Por ultimo mude a altura da nave apertando Shift,","mude para a nave vermelha e destrua os blocos vermelhos"])
		
	}
}


var justOneInTutorial
var wellDone=false

function playableTutorial(){
 if(actualScreen == constants.screens.tutorial3){
	 if(!timeInSec(6)) justOneInTutorial = 0
		
		if(timeInSec(6) && justOneInTutorial==0 && !wellDone){
		 justOneInTutorial++
			
			totalBlockID++
			
			blockBorn(0*8,0*8+(12*8),1,totalBlockID,1,"S")
   blockBorn(1*8,0*8+(12*8),1,totalBlockID,2,"S")
   blockBorn(0*8,1*8+(12*8),1,totalBlockID,3,"S")
   blockBorn(1*8,1*8+(12*8),1,totalBlockID,4,"S")
		}
		
		if(!wellDone)spr(330,0*8,3*8,0,1,0,0,1,2)
		
		var verify = 0
		for(var i =0;i<objects.length;i++){
		 if(objects[i].x<-8 && objects[i].y>3*8
			&& objects[i].y<4*8 && objects[i].type == constants.types.blockS){
			 verify++
			}
		}
		
		if(verify>=2){ 
		 wellDone=true
		 gameTotalTime=0
		}
			
		if(wellDone){
			print("MUITO BEM!!",12.8*8,11*8,1,false,1,true)
		 print("Aperte 'z' para continuar",10*8,12.5*8,1,false,1,true)
		 
			screenHuds=[]
			
			if(timeInSec(8) || btn(4)){ 
			 actualScreen++
				wellDone=false
			}
		}
	}
	
	if(actualScreen == constants.screens.tutorial4){
	 if(!timeInSec(6)) justOneInTutorial = 0
		
		if(timeInSec(6) && justOneInTutorial==0 && !wellDone){
		 justOneInTutorial++
			
			totalBlockID++
			
			blockBorn(0*8,0*8+(8*8),1,totalBlockID,1,"L")
   blockBorn(1*8,0*8+(8*8),1,totalBlockID,2,"L")
   blockBorn(2*8,0*8+(8*8),1,totalBlockID,3,"L")
   blockBorn(2*8,1*8+(8*8),1,totalBlockID,4,"L")
		}
		
		if(!wellDone)spr(323,0*8,8*8,0,1,0,0,1,3)
		
		var verify = 0
		for(var i =0;i<objects.length;i++){
		 if(objects[i].x<-8 && objects[i].y>8*8
			&& objects[i].y<10*8 && objects[i].type == constants.types.blockS
			&& objects[i].r == 3){
			 verify++
			}
		}
		
		if(verify>=2){ 
		 wellDone=true
		 gameTotalTime=0
		}
			
		if(wellDone){
			print("MUITO BEM!!",12.8*8,11*8,1,false,1,true)
		 print("Aperte 'z' para continuar",10*8,12.5*8,1,false,1,true)
		 
			screenHuds=[]
			
			if(timeInSec(8) || btn(4)){ 
			 actualScreen++
				wellDone=false
			}
		}
	}
	
	if(actualScreen == constants.screens.tutorial5){
	 if(!timeInSec(6)) justOneInTutorial = 0
		
		if(timeInSec(6) && justOneInTutorial==0 && !wellDone){
		 justOneInTutorial++
			
			totalBlockID++
			
			blockBorn(0*8,0*8+(4*8),1,totalBlockID,1,"R")
   blockBorn(1*8,0*8+(4*8),1,totalBlockID,2,"R")
   blockBorn(1*8,1*8+(4*8),1,totalBlockID,3,"R")
   blockBorn(2*8,1*8+(4*8),1,totalBlockID,4,"R")
		}
		
		if(!wellDone)spr(329,0*8,11*8,0,1,0,0,1,3)
		
		var verify = 0
		for(var i =0;i<objects.length;i++){
		 if(objects[i].x<-8 && objects[i].y>11*8
			&& objects[i].y<13*8 && objects[i].type == constants.types.blockS
			&& (objects[i].r == 1 || objects[i].r == 3)){
			 verify++
			}
		}
		
		if(verify>=2){ 
		 wellDone=true
		 gameTotalTime=0
		}
			
		if(wellDone){
			print("MUITO BEM!!",12.8*8,11*8,1,false,1,true)
		 print("Aperte 'z' para continuar",10*8,12.5*8,1,false,1,true)
		 
			screenHuds=[]
			
			if(timeInSec(8) || btn(4)){ 
			 actualScreen++
				wellDone=false
			}
		}
	}
	
	if(actualScreen == constants.screens.tutorial6){
	 if(!timeInSec(4)) justOneInTutorial = 0
		
		var Y = 4+Math.round(Math.random()*(12-4))
		
		if(timeInSec(4) && justOneInTutorial==0 && !wellDone){
		 justOneInTutorial++
			
			totalBlockID++
			
			blockBorn(0*8,0*8+(Y*8),2,totalBlockID,1,"R")
   blockBorn(1*8,0*8+(Y*8),2,totalBlockID,2,"R")
   blockBorn(1*8,1*8+(Y*8),2,totalBlockID,3,"R")
   blockBorn(2*8,1*8+(Y*8),2,totalBlockID,4,"R")
		}
		
		if(player.pts>=20){
			print("PARABENS",3*8,5*8,2,false,4)
   print("Aperte 'z' ou 'x' para voltar para o menu",6.8*8,12.5*8,1,false,1,true)
		 
			screenHuds=[]
			
			wellDone=true
			
			if(btn(4) || btn(5)){ 
			 actualScreen=constants.screens.menu
				wellDone=false
			}
		}
	}
	
	playerLose()
}

//////////////////////////////

var ArrowSelector = function(){

	var self = this
	
	this.cord=0
	this.r=0
	this.x=13.9*8
	this.y=13*8
	this.velocity=15
	
	this.draw = function(){
	 spr(496,self.x,self.y,0,1,self.r,0,1,1)
	}
	
	this.menuCord = [
	 [
		 [13.9,13,constants.screens.menu,constants.screens.title,0],
		],
		[
		 [6.4,12.4,constants.screens.tutorial1,constants.screens.title,0],
		 [11.7,12.4,constants.screens.survive,constants.screens.title,0],
		 [16.4,12.4,constants.screens.story,constants.screens.title,0],
		 [21.3,12.4,constants.screens.options,constants.screens.title,0],
		],
		[],
		[],
		[
		 [2.6,4,constants.screens.menu,constants.screens.menu,3],
			[10.7,4,constants.screens.menu,constants.screens.menu,3],
			[18,4,constants.screens.menu,constants.screens.menu,3],
			[25.9,4,constants.screens.menu,constants.screens.menu,3],
	  
			[14.4,7.9,constants.screens.menu,constants.screens.menu,3],
			
			[2.6,13,constants.screens.menu,constants.screens.menu,3],
			[10.7,13,constants.screens.menu,constants.screens.menu,3],
			[17.8,13,constants.screens.menu,constants.screens.menu,3],
			[25.9,13,constants.screens.menu,constants.screens.menu,3],
	 ],
		[
		 [14.4,10,constants.screens.menu,constants.screens.menu,1],
		],
		[
		 [14.4,10,constants.screens.menu,constants.screens.menu,2],
		],
		[
		 [14.4,10,constants.screens.menu,constants.screens.menu,0],
		],
		[
		 [13.8,12,constants.screens.tutorial2,constants.screens.menu,0],
		],
		[
		 [13.8,12.5,constants.screens.tutorial3,constants.screens.tutorial1,0],
		],
	] 
	
	this.move = function(){
		if(self.velocity>0)self.velocity--
		
		if(self.velocity==0){
		 if(btn(3)){
				self.cord++
				if(self.cord==self.menuCord[actualScreen].length){
				 self.cord=0
				}
				
				self.velocity=10
			}
			if(btn(2)){ 
				self.cord--
				if(self.cord==-1) self.cord = self.menuCord[actualScreen].length-1
				
			 self.velocity=10
			}
		}
		
		self.x = (self.menuCord[actualScreen][self.cord][0])*8
		self.y = (self.menuCord[actualScreen][self.cord][1])*8
	}
	
	this.select = function(){
		
		if(btn(4) && self.velocity==0){
		 actualScreen = self.menuCord[actualScreen][self.cord][2]
		 	
			self.velocity = 15
		 screenHuds=[]
			
			sfx(16,'C-6',10,0,8)
			
			self.cord=0
		}
		
		if(btn(5) && self.velocity==0){
		 var cord = self.cord
		
			self.cord = self.menuCord[actualScreen][cord][4]
		
		 actualScreen = self.menuCord[actualScreen][cord][3]
			
			sfx(17,'A#5',10,0,8)
			
			self.velocity = 15
		 screenHuds=[]
		}
	}
	
	this.att = function(){
	 self.draw()
		self.move()
		self.select()
	}
}

var arrowSelector = new ArrowSelector

////////////////////

function GameOver(){
 print("GAME OVER",2*8,5*8,2,false,4)
 print("(tente novamente apertanto 'z' ou 'x')",6.8*8,9*8,2,false,1,true)
 
	if(actualScreen == constants.screens.gameOver+2){
	 print("Sinceramente como voce morreu no tutorial?...",5.5*8,15*8,14,false,1,true)
	}
}

////////////////////

var gameTotalTime = 0
var actualScreen = constants.screens.title

function timeInSec(_s){
	
	GT = Math.floor(gameTotalTime/60)
	ST = _s*(Math.floor(gameTotalTime/60/_s))
	
	S = GT==ST  
	
	if(GT == 0 && ST == 0) S = false
	
	return S
}

function gameTimer(){
 gameTotalTime++
}

var ENGINE = [
														[worldBackground,
														 gameTitle,
															gameInfo,
										     arrowSelector.att],
				          
														[worldBackground,
														 gameTitle,
															gameInfo,
															justExplosionsAtt,
										     arrowSelector.att],
														
														
														[worldBackground,
															gameTimer,
														 worldObjects,
															worldSoundAndHud],
														
														
														[worldBackground,
															gameTimer,
														 worldObjects,
															worldSoundAndHud],
														
														[worldBackground,
															arrowSelector.att,
															options],
														
														
														[worldBackground,
															worldSoundAndHud,
															worldObjects,
															arrowSelector.att,
															GameOver],
															
														[worldBackground,
															worldSoundAndHud,
															worldObjects,
															arrowSelector.att,
															GameOver],
															
														[worldBackground,
															worldSoundAndHud,
															worldObjects,
															arrowSelector.att,
															GameOver],
															
														[worldBackground,
														 gameTitle,
															gameInfo,
															worldSoundAndHud,
										     arrowSelector.att],
															
														[worldBackground,
														 gameTitle,
															gameInfo,
															worldSoundAndHud,
										     arrowSelector.att],
														
														[gameTimer,
														 worldBackground,
														 worldObjects,
															playableTutorial,
															gameInfo,
															worldSoundAndHud],
														
														[gameTimer,
														 worldBackground,
														 worldObjects,
															playableTutorial,
															gameInfo,
															worldSoundAndHud],
														
														[gameTimer,
														 worldBackground,
														 worldObjects,
															playableTutorial,
															gameInfo,
															worldSoundAndHud],
														
														[gameTimer,
														 worldBackground,
														 worldObjects,
															playableTutorial,
															gameInfo,
															worldSoundAndHud],
															
													]

function TIC(){
	
	if(key(57)) reset()
	
 for(var i = 0;i<ENGINE[actualScreen].length;i++){
	 ENGINE[actualScreen][i]()	
	}
	
}

