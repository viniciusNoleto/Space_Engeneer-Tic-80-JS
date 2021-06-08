# Space-Engeneer-Tic-80-JS
Projeto pessoal: Jogo de nave misturado com Tetris

// title:  Pensar Sobre
// author: Vini
// desc:   Navinha + Tetris
// script: js

var up = 0
var down = 1
var left = 2
var right = 3

const constants={
 types:{
	 player:0,
	 blockS:1,
	 blockN:2,
		bullet:3,
	}
}

var objects=[]
var background=[]
var explosions=[]
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
	
	this.maxAmmo = [["missileLife",8+1],
	                ["freezeLife",4+1],
																	["nitroLife",10+1],
																	["rotateLife",20+1],
																	["tractorLife",6+1]]
	
	this.missileLife = self.maxAmmo[0][1]
 this.freezeLife = self.maxAmmo[1][1]
 this.nitroLife = self.maxAmmo[2][1]
 this.rotateLife = self.maxAmmo[3][1]
 this.tractorLife = self.maxAmmo[4][1]
 
	this.nextSprite = self.sprite
	
	this.addPts = function(_pts,_b){
	 _b = (typeof _b === 'undefined') ?
	  true : _b
		
		self.pts+=_pts
		if(_b){
		 sfx(13,'C-5',10,3,8)
	 }
	}
	
	this.hit = function(obj,self){
  
		if((obj.type==1 && self.height==1)
		|| (obj.type==2 && self.height==2)){
		 self.visible=false
			
			screenHuds=[]
			hudBorn( 40,["Meu Deus, eles morreram...","...","Alo base? Vamos precisar de novos coletores..."])
		 
			pixBorn(360,self.x,self.y,2,self.height,90)
		}
		
 }
	
	this.GUIA = function(self){
	
		spr(263 + self.sprite-1*(257+(16*(self.height-1))),
		    self.x-7,self.y-4,0,1,0,0,1,1)
	}
	
	this.move = function(self){
	 self.dy=0
	 self.dx=0
	 
		if(btn(up) && self.moveTimer==0){
			self.moveTimer = -4
		}
	 if(btn(down) && self.moveTimer==0){
		 self.moveTimer = 4
		}
		
		if(self.moveTimer>0){
			self.moveTimer--
			self.dy+=(8/4)
		}else if(self.moveTimer<0){
		 self.moveTimer++
			self.dy+=-(8/4)
		}
		
	 if(btn(left)) self.dx=-self.velocity
	 if(btn(right)) self.dx=+self.velocity
		
		self.x+=self.dx
  self.y+=self.dy
	}
	
	this.gun = function(self){
	 if(self.reload<15) self.reload++
		
		var bullets = [[Missile,Missile],
		               [Freeze,Freeze],
																	[Nitro,Nitro],
																	[RotateL,RotateR],
																	[Tractor,Tractor]]
		
		if(self.sprite>=257 && self.sprite!=272){
			if(key(1)){
				if(self.reload==15){
			  if(self[self.maxAmmo[self.sprite-1*(257
							 +(16*(self.height-1)))][0]]
						> 1){
						
							bulletBorn(
							 bullets[self.sprite-1*(257
								 +(16*(self.height-1)))][0],
							 self.x,self.y,self.height
							)
					}else{
					 self[self.maxAmmo[self.sprite-1*(257
							 +(16*(self.height-1)))][0]]=-1 
					}
					
				 sfx(10,'C-2',10,1,10)
				}
				self.reload=0
			}
			
			if(key(4)){
			 if(self.reload==15){
			  if(self[self.maxAmmo[self.sprite-1*(257
							 +(16*(self.height-1)))][0]]
					 > 1){
							
							bulletBorn(
							 bullets[self.sprite-1*(257
								 +(16*(self.height-1)))][1],
							 self.x,self.y,self.height
							)
				 }else if(self[self.maxAmmo[self.sprite-1*(257
							 +(16*(self.height-1)))][0]]==1){
					 self[self.maxAmmo[self.sprite-1*(257
							 +(16*(self.height-1)))][0]]=-1 
					}
					
					sfx(10,'C-2',10,1,10)
				}
				
				self.reload=0
			}
		}
	}
	
	this.switchShip = function(self){
		if(self.transform<20){
		 self.transform++
		}
		
		if(key(23)){
		 if(self.transform>=14){
			 self.nextSprite--
				
				if(self.nextSprite==256+(16*(self.height-1))) self.nextSprite=261+(16*(self.height-1))
				
				self.transform=0
				
				sfx(11,'G-3',10,1,10)
			
			 for(var i = 0;i<5;i++){
				 if(self[self.maxAmmo[i][0]] < self.maxAmmo[i][1]){
					 self[self.maxAmmo[i][0]]+=2
					}
	   }
			}
		}
		
		if(key(19)){
		 if(self.transform>=14){
			 self.nextSprite++
				
				if(self.nextSprite==262+(16*(self.height-1))) self.nextSprite=257+(16*(self.height-1))
				
				self.transform=0
				
				sfx(11,'G-3',10,1,10)
				
				for(var i = 0;i<5;i++){
				 if(self[self.maxAmmo[i][0]] < self.maxAmmo[i][1]){
					 self[self.maxAmmo[i][0]]+=2
					}
	   }
			}
		}
		
		if(key(64)){
		 if(self.transform==20){
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
		}
		
		if(self.transform<=8){
		 self.sprite = 256+(16*(self.height-1))
		}else{
		 self.sprite = self.nextSprite
		}
		
	}
	
	this.actions = function(self){
		self.switchShip(self)
		self.gun(self)
		self.GUIA(self)
	}
	
}

var player = new Player()
objects.push(player)

/////////

var Bullet = function(_sprite,_color,_x,_y,_height){
 
	ObjectModel.call(
	 this,_sprite,_height,_x+2,_y-4,
		0,1,1,1,4,constants.types.bullet
	)
	
	this.colli = [-(3),(3)-1,
	              -(3),(3)-1]
	this.boom
	
	this.actions = function(self){
	 if(self.x >=300) bulletsCount++ 
	}
	
	this.hit = function(obj,self){
		if((obj.type == constants.types.blockS && self.height==1)
		|| (obj.type == constants.types.blockN && self.height==2)){
		 for(var i=0;i<objects.length;i++){
	   if(objects[i].ID == obj.ID){
		   self.boom(self,objects[i])
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
	
	player.missileLife--
	
	this.boom = function(self,obj){
	 obj.visible=false
		var c
		if(obj.type==constants.types.blockN){
		 c=14
			player.addPts(1,false)
		}else{
		 c=7
			if(obj.n==1){
			 hudBorn( 12,["","POR QUE VOCE EXPLODIU UM BLOCO VERDE SUA ANTA??"])
		  player.lifes--
				player.anger+=3
			}
			player.addPts(-5,false)
		}
		sfx(12,'D-2',15,2,10)
		
		pixBorn(50,obj.x,obj.y,c,obj.height,24)
	}
}

var Freeze = function(_x,_y,_height){
 Bullet.call(
		this,289,11,_x,_y,_height
	)
	
	player.freezeLife--
	
	this.boom = function(self,obj){
	 obj.freeze=true
		obj.freezeTimer=60*3
		obj.sprite=34
	}
}

var Nitro = function(_x,_y,_height){
 Bullet.call(
		this,290,8,_x,_y,_height
	)
	
	player.nitroLife--
	
	this.boom = function(self,obj){
	 if(obj.type == constants.types.blockN){
		 obj.velocity = 1.4
		}else{
		 obj.velocity = 1
		}
		
		obj.nitroTimer=60*2
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
		 player.anger++
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
	
	player.rotateLife--
	
	var X,Y,R
	var rotated = false
	
	this.boom = function(self,obj){
	 
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
	
	player.rotateLife--
	
 var X,Y,R
	var rotated = false
	
	this.boom = function(self,obj){
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
	
	player.tractorLife--
	
	var self = this
	
	this.draw = function(self){
	 line(player.x+5,Math.round(player.y)-4,self.x,self.y,13)
	 line(player.x+5,Math.round(player.y)+3,self.x,self.y,13)
	}
	
	this.boom = function(self,obj){
	 if(!obj.grabbed){
			obj.freeze=true
			obj.grabbed=true
			
			obj.grabbDY=player.y-obj.y
			obj.freezeTimer=12
			obj.grabbTimer=60*4
			
			if(obj.type == constants.types.blockN){
		  obj.sprite=49
		 }else{
		  obj.sprite=48
		 }
	 }else{
			obj.grabbed=false
		}
	}
}

function bulletBorn(TYPE,_x,_y,_height){
 
	var bullet = new TYPE(_x,_y,_height)
										
 objects.push(bullet)
}

/////////

var Block = function(_x,_y,_type,_ID,_n,_format){
 
	var v
	
	if(_type==constants.types.blockN){
	 v=0.8
	}else{
	 v=0.4
	}
	
	ObjectModel.call(
	 this,31+_type,_type,_x+240,_y,
		180,1,1,1,v,_type
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
	
	this.actions = function(self){
		if(self.freezeTimer>=2){
		 self.freezeTimer--
		}else if(self.freezeTimer==1){
		 self.freeze=false
			
			if(self.type == constants.types.blockN){
			 self.sprite=33
			}else{
			 self.sprite=32
			}
			
			self.freezeTimer--
		}
		if(self.nitroTimer>=2){
		 self.nitroTimer--
		}else if(self.nitroTimer==1){
		 if(self.type == constants.types.blockN){
			 self.velocity = 0.8
			}else{
			 self.velocity = 0.4
			}
			
			self.nitroTimer--
		}
		if(self.grabbed){
		 if(self.y < player.y-self.grabbDY){
				self.y++
			}
			if(self.y > player.y-self.grabbDY){
			 self.y--
			}
			
			if(self.grabbTimer==0){
			 self.grabbed=false
			}
			
			self.grabbTimer--
			self.freezeTimer=12
		}
	}
	
	this.hit = function(obj,self){
	 if(obj.height == self.height
		&&(obj.type == constants.types.blockN
		|| obj.type == constants.types.blockS)
		&& obj.ID != self.ID){
			for(var i=1;i<objects.length;i++){
			 if(objects[i].ID == self.ID
				|| objects[i].ID == obj.ID){
					var c
				 
				 if(objects[i].height==1){
					 c=7
						player.addPts(-6)
				 }else{
				  c=14
						player.addPts(2)
					}
					
			  pixBorn(50,objects[i].x,objects[i].y,c,objects[i].height,24)
		  
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


/////////

var totalBlockID = 0
var blockT = 0
var blockSTimer=0

function blobckBornEngine(){
 if(blockT==60*2){
	 
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


////////

var Pix = function(_x,_y,_life,_color,_height){
	
	var self = this
			
	this.x=_x
	this.y=_y
	this.life=_life
	this.color=_color
	this.visible=true
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
	 	
		explosions.push(PIX)
	}
}

////////

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
			 sfx(14,'A#4',15,2,2)
				
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
		 
			player.anger++
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
var broked = false

function talkEvents(){
 if(bulletsCount==5){
	 
		 hudBorn( 12,["Nao gaste municoes assim,","o Imperio paga caro por elas!"])
  if(player.anger<5){
		 hudBorn( 44,["OK...","Vou tentar melhorar"])
	 }else if(player.anger>=5){
		 hudBorn( 76,["Vem aqui entao oh bonzao","Pilota e atira pra ver se e facil"])
  }
		
		bulletsCount=0
		player.anger+=2
	
	}
	
	const shipCheck=[["missileLife","missil"],
	                 ["freezeLife","tiro gelado"],
	                 ["nitroLife","tiro acelerador"],
																		["rotateLife","tiro rotacionador"],
									         ["tractorLife","raio trator"],]
									
	for(var i = 0;i<5;i++){
	 if(player[shipCheck[i][0]]==-1){
		 hudBorn(140,["Talvez... So talvez...", 
			 "O maquinario do "+shipCheck[i][1]+" esteja...","pegando fogo."])
		 
			broked=true
			player.anger++
			
			player[shipCheck[i][0]]++
		}
	}
	
	var allOk = 0
	if(broked){
		for(var i = 0;i<5;i++){
		 if(player[shipCheck[i][0]] == player.maxAmmo[i][1]){
		  allOk++
			}
		 if(allOk==5){
			 hudBorn(108,["Finalmente achei minha chave da sorte!!","Foi bem rapidinho pra arrumar :)"])
		  hudBorn( 76,["Espera, voce demorou tudo isso","so pra achar sua chave???","Por que voce nao usou outra?"])
		  hudBorn(108,["Claro??","E minha chave da sorte, so uso ela!!!"])
		  if(player.anger<3){
				 hudBorn( 72,["Coloca esse negocio do teu lado pelo amor de Deus"])
		  }else{
				 hudBorn( 76,["Eu juro que da proxima vez vou enfiar essa chave","no seu rabo pra voce achar mais rapido..."])
			  hudBorn(168,["..."])
				}
				broked=false
			}
		}
	}
}

var angerTimer=0

function calmDown(){
 if(player.anger > 0){
	 angerTimer++
	}else{
	 angerTimer=0
	}
	
	if(angerTimer/60==1*60){
	 if(player.anger > 0) player.anger--
 }
}

function scoreboard(){
 spr(296,12.5*8,0,0,1,0,0,3,1)
	print(Math.round(player.pts),1+12.5*8,2,1)
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
					player.lifes-=0.5
					
					objects[j].pointEarned=true
				}
				
			}
	 }
	 if(destroy){
	  wayToBlock[i][3]= false
		}
	}
	
}

////

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
var END = false

function playerLifes(){
 var playerInGame = false
	for(var i = 0;i<objects.length;i++){
	 if(objects[i].type == constants.types.player) playerInGame=true 
	}
	
	if(player.lifes <=0 || !playerInGame){
	 if(player.lifes <=0){
		 if(!fired){
			 hudBorn( 12,["Voce e um pessimo coletor esta DEMITIDO!!"])
			}
			fired=true
	 }
		
		for(var i = 0;i<objects.length;i++){
	  if(objects[i].type == constants.types.player) objects.splice(i,1)
	 }
		if(!END){
		 pixBorn(360,player.x,player.y,2,player.height,90)
	  gameTotalTime=0
		}
		
		END = true
	}
	
	if(END){
	 if(timeInSec(5)){
		 actualScreen = 3
		}
	}
 
}


////

var bossTime = false
var finalDialogue = true
var level = 1
var foundationX = 240+24
var bossSprite = 0
var credits = 0

function gameEngine(){
 if(timeInSec(60*5)) bossTime=true
	
	if(!bossTime){
	 if(!END){
			wayToGoEngine()
	  blobckBornEngine()
	 }
		playerLifes()
	}else{
		if(level<=3){
		 if(finalDialogue){
			 gameTotalTime=0
			}
			finalDialogue=false
			
			foundationX-=0.5
			
			bossSprite+=0.1
			if(bossSprite>=2) bossSprite=0
			
			spr(64+((level-1)*48)+Math.floor(bossSprite)*3,
			    foundationX,7*8,0,1,0,0,3,3)
			spr(208+((level-1)*16),
			    foundationX-18,8*8,0,1,0,0,2,1)
			print("Parabens coletores!!",
			foundationX,5*8,1,false,1,true)
			print("Voces contruibuiram com o Imperio!",
			foundationX,6*8,1,false,1,true)
			
			print("aperte 'z' para pular",10.5*8,14*8,2,false,1,true)
			
			if(timeInSec(14) || btn(4)){
			 foundationX = 240+24
				bossTime=false
				finalDialogue = true
				level++
			}
		}else{
		 if(finalDialogue){
			 hudBorn(  8,["Obrigado, sem voce eu nunca seria promovido :)","...","Mas isso nao quer dizer que seu salario vai aumentar."])
			}
			finalDialogue = false
			
			print("Em breve mais atualizacoes...",0,credits-48,1)
			print("GitHub: viniciusNoleto",0,credits-24,1)
		 print("Desenvolvedor: Vinicius Noleto de Araujo",0,credits-16,1)
		 print("Muito obrigado por jogar :)",0,credits,1)
		 
			credits+=0.1
		}
	}
}

////////


function hudEngine(){
 if(!bossTime){
		randomTalks()
		talkEvents()
		calmDown()
 }
}

////////


function objAtt(array){
 for(var H=0;H<=3;H++){
	 for(var i=0;i<array.length;i++){
		 if(array[i].height==H)array[i].att()
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

function worldObjects(){
 gameEngine()
	
	objAtt(objects)
	
	objAtt(explosions)
		
	objEliminate(objects,-24,300)
	
	objEliminate(explosions,-24,300)
}

function worldSoundAndHud(){
 scoreboard()
	
	if(actualScreen==0)hudEngine()
	
	hudAtt()
	
	objAtt(soundEffects)
	
	notVisibleEliminate(soundEffects)
	
	notVisibleEliminate(screenHuds)
}

////////////////////

function gameTitle(){
	spr(448,9*8,3*8,0,2,0,0,6,4)
}

function gameInfo(){
 if(actualScreen == 1){
	 print("aperte 'z' para continuar",9*8,12*8,2,false,1,true)
	}else if(actualScreen == 2){
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
	}
}

var tutorialRead=false

var Coin = function(){
 
	var self = this
	
	this.cord=0
	this.x=13.5
	this.y=13
	this.velocity=15
	
	this.draw = function(){
	 spr(416,self.x*8,self.y*8,0,1,0,0,1,1)
	}
	
	//this.menuCord = [[self.x*8,self.y*8],[2,0]] 
	
	this.move = function(){
		if(self.velocity>0)self.velocity--
		
		/*
		if(self.velocity==0){
		 if(btn(right)){
				self.cord++
				if(self.cord==2)self.cord=1
				self.velocity=10
			}
			if(btn(left)){ 
				self.cord--
				if(self.cord==-1)self.cord=0
			 self.velocity=10
			}
		}else{
		 self.velocity--
		}
		
		self.x = self.menuCord[self.cord][0]
		self.y = self.menuCord[self.cord][1]
	 */
	}
	
	this.select = function(){
	 
		if(btn(4) && self.velocity==0
		&& actualScreen==1){
		 actualScreen=2
			self.velocity = 10
			self.y-=1
		}else if(btn(4) && self.velocity==0
		&& actualScreen==2){
		 actualScreen=0
			screenHuds=[]
		}
	}
	
	this.att = function(){
	 self.draw()
		self.move()
		self.select()
	}
}

var coin = new Coin

////////////////////

function GameOver(){
 print("GAME OVER",2*8,5*8,2,false,4)
 print("(tente novamente apertanto 'z')",7.5*8,9*8,2,false,1,true)
 
	if(btn(4)) reset()
}

////////////////////

var gameTotalTime = 0
var actualScreen = 1

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
              [gameTimer,
														 worldBackground,
														 worldObjects,
															worldSoundAndHud],
														
														[worldBackground,
														 gameTitle,
															gameInfo,
										     coin.att],
															
														[worldBackground,
														 gameInfo,
															worldSoundAndHud,
															coin.att],
													 
														[worldBackground,
															worldSoundAndHud,
															worldObjects,
															GameOver]
													]

function TIC(){
	
 for(var i = 0;i<ENGINE[actualScreen].length;i++){
	 ENGINE[actualScreen][i]()	
	}

}
