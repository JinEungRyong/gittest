var config = {
  base: { value: 1000, type: 'balance', label: 'base bet' }
  basePayout: { value: 1.1, type: 'multiplier', label: 'base payout' }
  stop: { value: 60, type: 'multiplier', label: 'base payout' }
  loss: { value: 'increase', type: 'radio', label: 'On loss',
        options: { base: { type: 'noop', label: 'return to base payout'},
		increase: { value: 1, type: 'multiplier', label: 'increase payout' },
		}
		},
  win: { value:'base', type:'radio', label:'On win',
        options: { base: { type: 'noop', label: 'return to base payout'},
		increase: { value:1, type:'multiplier', label: 'increase payout'},
		}
  },
};

log('Script is running..');

var currentPayout = config.basePayout.value;
var currentBase = config.base.value;
var goal = 1100;

engine.on('GAME_STARTING',onGameStarted);
engine.on('GAME_ENDED',onGameEnded);

function onGameStarted() {
	engine.bet(currentBase,currentPayout);
}

function onGameEnded(info){
	var lastGame = engine.history.first();
	
	if(!lastGame.wager){
		return;
	}
	
	if(lastGame.cashedAt){
		if(config.win.value ==='base'){
			currentPayout = config.basePayout.value;
			currentBase = config.base.value;
			goal = 1100;
			log('Win base:',Math.round(currentBase/100),' payout:',currentPayout);
		}else {
			console.assert(config.win.value === 'increase');
			currentPayout += config.win.options.increase.value;
		}
	}else{
		if(config.loss.value === 'base'){
			currentPayout = config.basePayout.value;
		}else {
			console.assert(config.loss.value === 'increase');
			currentPayout += config.loss.options.increase.value;
			goal += 1000;
			cal_bet();
			log('loss base:',Math.round(currentBase/100),' payout:',currentPayout, 'goal:',goal);
		}
	}
	
	if( currentPayout > config.stop.value){
		currentPayout = config.basePayout.value;
		currentBase = config.base.value;
		goal = 1100;
		log('stop base:',Math.round(currentBase/100),' payout:',currentPayout, 'goal:',goal);
	}
}

function cal_bet(){
	currentBase = Math.round(goal/currentPayout);
	
}
