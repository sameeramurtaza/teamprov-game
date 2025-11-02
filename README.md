# teamprov-game
Step into TeamProv, a lively virtual cooperative improv game where eight players unite to craft a hilarious, unpredictable story. Choose quirky characters, build scenes together, and adapt to wild twists introduced by the facilitator. Through laughter, creativity, and teamwork, players collaborate to weave chaos into a shared, unforgettable tale.
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TeamProv - Virtual Improv Experience</title>
    <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect } = React;

        const TeamProvGame = () => {
          const [gameState, setGameState] = useState('landing');
          const [roomCode, setRoomCode] = useState('');
          const [playerName, setPlayerName] = useState('');
          const [players, setPlayers] = useState([]);
          const [currentPlayer, setCurrentPlayer] = useState(null);
          const [selectedCharacter, setSelectedCharacter] = useState(null);
          const [selectedWord, setSelectedWord] = useState(null);
          const [selectedScene, setSelectedScene] = useState(null);
          const [story, setStory] = useState([]);
          const [currentRound, setCurrentRound] = useState(1);
          const [timeLeft, setTimeLeft] = useState(480);
          const [currentTurn, setCurrentTurn] = useState(0);
          const [facilitatorMessage, setFacilitatorMessage] = useState('');
          const [storyInput, setStoryInput] = useState('');
          const [currentTwist, setCurrentTwist] = useState(null);
          const [currentConstraint, setCurrentConstraint] = useState(null);

          const characters = [
            "Grandma on a motorcycle", "Pirate dentist", "Alien barista", "Nervous superhero",
            "Talking cat", "Zookeeper", "Chef that makes everything out of cheese", "Toddler CEO",
            "Taxi driver who drives a limo", "Onion farmer", "Giraffe veterinarian", "Robot",
            "Accountant", "Blimp pilot", "Plumber", "Ghost of Elvis"
          ];

          const randomWords = [
            "Sandwich", "Volcano", "Glitter", "Lost sock", "Pickle", "Umbrella", "Secret potion",
            "Pogo stick", "Karaoke", "Disco ball", "I ❤️ NY t-shirt", "Diet coke", "Deodorant",
            "Waffle maker", "Construction hat", "XXXL pizza", "Haunted microphone"
          ];

          const scenes = [
            "Haunted office", "Elementary school science fair", "Lost at the zoo", "New York at Christmas",
            "Renaissance fair", "Airport during a snowstorm", "Amusement park after closing time",
            "Space station malfunction", "Mountain cabin in a thunderstorm", "Farmer's market",
            "Subway train", "Aquarium after dark", "Grocery store power outage", "School picture day",
            "TV game show", "New York City marathon", "Olympics"
          ];

          const twists = [
            "Someone turns into a chicken", "Time portal opens up", "Two characters switch bodies",
            "A celebrity suddenly arrives", "The floor is lava", "Someone finds a treasure map",
            "A loud alarm goes off", "Someone reveals they are an undercover agent",
            "Everyone must rhyme for a round", "Everyone realizes they're in a dream",
            "Someone discovers they have an evil twin", "A secret door appears",
            "Someone can only speak in questions", "The story jumps 10 years in the future"
          ];

          const constraints = [
            "One person can only speak in questions", "One person can only use one-syllable words",
            "One person must repeat the last word of every sentence", "One person must sing their lines",
            "One person has to have sentences that start alphabetically", "One person has to speak in slow motion",
            "One person can only speak using sound effects", "One player can only whisper",
            "One person can only talk about food", "One person must begin every sentence with 'Actually…'",
            "One person can only speak in metaphors", "No one can say 'yes' or 'no'",
            "Each player has to incorporate an object nearby in their sentence"
          ];

          const generateRoomCode = () => {
            return Math.random().toString(36).substring(2, 8).toUpperCase();
          };

          const createRoom = () => {
            const code = generateRoomCode();
            setRoomCode(code);
            setGameState('lobby');
            setFacilitatorMessage(`Welcome to TeamProv! Room code: ${code}. Share this code with your team members so they can join!`);
          };

          const joinRoom = () => {
            if (roomCode && playerName) {
              setGameState('lobby');
              setFacilitatorMessage(`Welcome ${playerName}! Waiting for other players to join...`);
            }
          };

          const addPlayer = () => {
            if (playerName && !players.find(p => p.name === playerName)) {
              const newPlayer = { name: playerName, character: null, word: null };
              setPlayers([...players, newPlayer]);
              setCurrentPlayer(newPlayer);
              setFacilitatorMessage(`${playerName} has joined! Total players: ${players.length + 1}`);
            }
          };

          const startCharacterSelection = () => {
            setGameState('characterSelect');
            setFacilitatorMessage("Time to pick your characters and random words! Each player must select one character and one word to use in the story.");
          };

          const selectCharacter = (char) => {
            setSelectedCharacter(char);
          };

          const selectWord = (word) => {
            setSelectedWord(word);
          };

          const confirmSelection = () => {
            if (selectedCharacter && selectedWord) {
              const updatedPlayers = players.map(p => 
                p.name === playerName ? { ...p, character: selectedCharacter, word: selectedWord } : p
              );
              setPlayers(updatedPlayers);
              setFacilitatorMessage(`Great choice, ${playerName}! You'll be playing as: ${selectedCharacter} with the word: ${selectedWord}`);
            }
          };

          const startGame = () => {
            const scene = scenes[Math.floor(Math.random() * scenes.length)];
            setSelectedScene(scene);
            setGameState('playing');
            setCurrentRound(1);
            setTimeLeft(480);
            setFacilitatorMessage(`Round 1: Building the Story! Scene: ${scene}. ${players[0]?.name}, you're up first! Add 2 sentences using your character (${players[0]?.character}) and word (${players[0]?.word}).`);
          };

          const submitStory = () => {
            if (storyInput.trim()) {
              const newEntry = {
                player: playerName,
                text: storyInput,
                round: currentRound
              };
              setStory([...story, newEntry]);
              setStoryInput('');
              
              const nextTurn = (currentTurn + 1) % players.length;
              setCurrentTurn(nextTurn);
              
              if (nextTurn === 0 && currentRound === 1) {
                setCurrentRound(2);
                introduceTwist();
              } else {
                setFacilitatorMessage(`Nice! ${players[nextTurn]?.name}, your turn! Build on the story.`);
              }
            }
          };

          const introduceTwist = () => {
            const twist = twists[Math.floor(Math.random() * twists.length)];
            setCurrentTwist(twist);
            setGameState('twist');
            setFacilitatorMessage(`PLOT TWIST! ${twist}. Continue the story incorporating this twist!`);
            setTimeout(() => {
              setGameState('playing');
            }, 5000);
          };

          const startFinale = () => {
            const constraint = constraints[Math.floor(Math.random() * constraints.length)];
            setCurrentConstraint(constraint);
            setCurrentRound(3);
            setGameState('finale');
            setFacilitatorMessage(`Round 3: The Grand Finale! Constraint: ${constraint}. Wrap up the story in an epic way!`);
          };

          const endGame = () => {
            setGameState('debrief');
            setFacilitatorMessage("Amazing work, team! You've created a masterpiece of improvisation. Let's reflect on the experience!");
          };

          useEffect(() => {
            if (gameState === 'playing' && timeLeft > 0) {
              const timer = setTimeout(() => setTimeLeft(timeLeft - 1), 1000);
              return () => clearTimeout(timer);
            }
          }, [timeLeft, gameState]);

          const Icon = ({ type, className }) => {
            const icons = {
              sparkles: <svg className={className} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M5 3v4M3 5h4M6 17v4m-2-2h4m5-16l2.286 6.857L21 12l-5.714 2.143L13 21l-2.286-6.857L5 12l5.714-2.143L13 3z" /></svg>,
              play: <svg className={className} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M14.752 11.168l-3.197-2.132A1 1 0 0010 9.87v4.263a1 1 0 001.555.832l3.197-2.132a1 1 0 000-1.664z" /><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M21 12a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>,
              users: <svg className={className} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197M13 7a4 4 0 11-8 0 4 4 0 018 0z" /></svg>,
              userPlus: <svg className={className} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M18 9v3m0 0v3m0-3h3m-3 0h-3m-2-5a4 4 0 11-8 0 4 4 0 018 0zM3 20a6 6 0 0112 0v1H3v-1z" /></svg>,
              clock: <svg className={className} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>,
              message: <svg className={className} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" /></svg>,
              zap: <svg className={className} fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M13 10V3L4 14h7v7l9-11h-7z" /></svg>
            };
            return icons[type] || null;
          };

          if (gameState === 'landing') {
            return (
              <div className="min-h-screen bg-gradient-to-br from-purple-600 via-pink-500 to-orange-400 flex items-center justify-center p-4">
                <div className="bg-white rounded-3xl shadow-2xl p-8 max-w-2xl w-full">
                  <div className="text-center mb-8">
                    <div className="flex justify-center mb-4">
                      <Icon type="sparkles" className="w-16 h-16 text-purple-600" />
                    </div>
                    <h1 className="text-5xl font-bold text-gray-800 mb-2">TeamProv</h1>
                    <p className="text-xl text-gray-600">Virtual Improv Storytelling Experience</p>
                  </div>
                  
                  <div className="bg-purple-50 rounded-xl p-6 mb-6">
                    <h2 className="text-2xl font-bold text-purple-800 mb-3">How to Play</h2>
                    <ul className="space-y-2 text-gray-700">
                      <li className="flex items-start">
                        <span className="text-purple-600 mr-2">•</span>
                        <span>5-8 players collaborate to build a hilarious story</span>
                      </li>
                      <li className="flex items-start">
                        <span className="text-purple-600 mr-2">•</span>
                        <span>Pick characters and random words to use in the story</span>
                      </li>
                      <li className="flex items-start">
                        <span className="text-purple-600 mr-2">•</span>
                        <span>Add 2 sentences per turn, building on others' ideas</span>
                      </li>
                      <li className="flex items-start">
                        <span className="text-purple-600 mr-2">•</span>
                        <span>Navigate plot twists and creative constraints</span>
                      </li>
                      <li className="flex items-start">
                        <span className="text-purple-600 mr-2">•</span>
                        <span>25 minutes of creativity, laughter, and teamwork!</span>
                      </li>
                    </ul>
                  </div>

                  <div className="space-y-4">
                    <input
                      type="text"
                      placeholder="Enter your name"
                      value={playerName}
                      onChange={(e) => setPlayerName(e.target.value)}
                      className="w-full px-4 py-3 border-2 border-purple-300 rounded-xl focus:outline-none focus:border-purple-500"
                    />
                    
                    <button
                      onClick={() => {
                        if (playerName) {
                          createRoom();
                          addPlayer();
                        }
                      }}
                      className="w-full bg-purple-600 text-white py-4 rounded-xl font-bold text-lg hover:bg-purple-700 transition flex items-center justify-center"
                    >
                      <Icon type="play" className="w-5 h-5 mr-2" /> Create New Room
                    </button>

                    <div className="flex gap-2">
                      <input
                        type="text"
                        placeholder="Enter room code"
                        value={roomCode}
                        onChange={(e) => setRoomCode(e.target.value.toUpperCase())}
                        className="flex-1 px-4 py-3 border-2 border-purple-300 rounded-xl focus:outline-none focus:border-purple-500"
                      />
                      <button
                        onClick={() => {
                          if (playerName && roomCode) {
                            joinRoom();
                            addPlayer();
                          }
                        }}
                        className="bg-pink-600 text-white px-6 py-3 rounded-xl font-bold hover:bg-pink-700 transition"
                      >
                        Join
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            );
          }

          if (gameState === 'lobby') {
            return (
              <div className="min-h-screen bg-gradient-to-br from-purple-600 via-pink-500 to-orange-400 p-4">
                <div className="max-w-4xl mx-auto">
                  <div className="bg-white rounded-3xl shadow-2xl p-8">
                    <div className="text-center mb-6">
                      <h1 className="text-4xl font-bold text-gray-800 mb-2">Room: {roomCode}</h1>
                      <p className="text-gray-600">Share this code with your team!</p>
                    </div>

                    <div className="bg-purple-100 rounded-xl p-6 mb-6">
                      <div className="flex items-start">
                        <Icon type="message" className="w-6 h-6 text-purple-600 mr-3 mt-1 flex-shrink-0" />
                        <p className="text-gray-800">{facilitatorMessage}</p>
                      </div>
                    </div>

                    <div className="bg-gray-50 rounded-xl p-6 mb-6">
                      <h3 className="text-xl font-bold text-gray-800 mb-4 flex items-center">
                        <Icon type="users" className="w-5 h-5 mr-2" /> Players ({players.length})
                      </h3>
                      <div className="grid grid-cols-2 gap-3">
                        {players.map((player, idx) => (
                          <div key={idx} className="bg-white p-3 rounded-lg shadow flex items-center">
                            <div className="w-10 h-10 bg-purple-500 rounded-full flex items-center justify-center text-white font-bold mr-3">
                              {player.name[0]}
                            </div>
                            <span className="font-semibold">{player.name}</span>
                          </div>
                        ))}
                      </div>
                    </div>

                    <div className="space-y-3">
                      <input
                        type="text"
                        placeholder="Add another player name"
                        value={playerName}
                        onChange={(e) => setPlayerName(e.target.value)}
                        className="w-full px-4 py-3 border-2 border-purple-300 rounded-xl focus:outline-none focus:border-purple-500"
                      />
                      <button
                        onClick={addPlayer}
                        className="w-full bg-pink-600 text-white py-3 rounded-xl font-bold hover:bg-pink-700 transition flex items-center justify-center"
                      >
                        <Icon type="userPlus" className="w-5 h-5 mr-2" /> Add Player
                      </button>
                      
                      {players.length >= 5 && (
                        <button
                          onClick={startCharacterSelection}
                          className="w-full bg-purple-600 text-white py-4 rounded-xl font-bold text-lg hover:bg-purple-700 transition flex items-center justify-center"
                        >
                          <Icon type="play" className="w-5 h-5 mr-2" /> Start Game
                        </button>
                      )}
                      {players.length < 5 && (
                        <p className="text-center text-gray-600">Need at least 5 players to start</p>
                      )}
                    </div>
                  </div>
                </div>
              </div>
            );
          }

          if (gameState === 'characterSelect') {
            return (
              <div className="min-h-screen bg-gradient-to-br from-purple-600 via-pink-500 to-orange-400 p-4">
                <div className="max-w-6xl mx-auto">
                  <div className="bg-white rounded-3xl shadow-2xl p-8">
                    <div className="bg-purple-100 rounded-xl p-6 mb-6">
                      <div className="flex items-start">
                        <Icon type="message" className="w-6 h-6 text-purple-600 mr-3 mt-1 flex-shrink-0" />
                        <p className="text-gray-800">{facilitatorMessage}</p>
                      </div>
                    </div>

                    <div className="grid md:grid-cols-2 gap-6 mb-6">
                      <div>
                        <h3 className="text-2xl font-bold text-gray-800 mb-4">Pick Your Character</h3>
                        <div className="grid grid-cols-2 gap-2 max-h-96 overflow-y-auto">
                          {characters.map((char, idx) => (
                            <button
                              key={idx}
                              onClick={() => selectCharacter(char)}
                              className={`p-3 rounded-lg text-sm font-semibold transition ${
                                selectedCharacter === char
                                  ? 'bg-purple-600 text-white'
                                  : 'bg-gray-100 text-gray-800 hover:bg-purple-100'
                              }`}
                            >
                              {char}
                            </button>
                          ))}
                        </div>
                      </div>

                      <div>
                        <h3 className="text-2xl font-bold text-gray-800 mb-4">Pick Your Random Word</h3>
                        <div className="grid grid-cols-2 gap-2 max-h-96 overflow-y-auto">
                          {randomWords.map((word, idx) => (
                            <button
                              key={idx}
                              onClick={() => selectWord(word)}
                              className={`p-3 rounded-lg text-sm font-semibold transition ${
                                selectedWord === word
                                  ? 'bg-pink-600 text-white'
                                  : 'bg-gray-100 text-gray-800 hover:bg-pink-100'
                              }`}
                            >
                              {word}
                            </button>
                          ))}
                        </div>
                      </div>
                    </div>

                    {selectedCharacter && selectedWord && (
                      <div className="bg-green-50 border-2 border-green-300 rounded-xl p-4 mb-4">
                        <p className="text-center text-green-800 font-semibold">
                          You selected: {selectedCharacter} with {selectedWord}
                        </p>
                      </div>
                    )}

                    <div className="flex gap-3">
                      <button
                        onClick={confirmSelection}
                        disabled={!selectedCharacter || !selectedWord}
                        className="flex-1 bg-purple-600 text-white py-4 rounded-xl font-bold text-lg hover:bg-purple-700 transition disabled:bg-gray-300 disabled:cursor-not-allowed"
                      >
                        Confirm Selection
                      </button>
                      <button
                        onClick={startGame}
                        className="flex-1 bg-green-600 text-white py-4 rounded-xl font-bold text-lg hover:bg-green-700 transition flex items-center justify-center"
                      >
                        <Icon type="zap" className="w-5 h-5 mr-2" /> Begin Story!
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            );
          }

          if (gameState === 'playing' || gameState === 'twist' || gameState === 'finale') {
            return (
              <div className="min-h-screen bg-gradient-to-br from-purple-600 via-pink-500 to-orange-400 p-4">
                <div className="max-w-6xl mx-auto">
                  <div className="bg-white rounded-3xl shadow-2xl p-8">
                    <div className="flex justify-between items-center mb-6">
                      <div>
                        <h2 className="text-3xl font-bold text-gray-800">
                          Round {currentRound}: {currentRound === 1 ? 'Building' : currentRound === 2 ? 'The Twist' : 'Grand Finale'}
                        </h2>
                        <p className="text-gray-600">Scene: {selectedScene}</p>
                      </div>
                      <div className="flex items-center bg-purple-100 px-4 py-2 rounded-lg">
                        <Icon type="clock" className="w-5 h-5 text-purple-600 mr-2" />
                        <span className="font-bold text-purple-800">{Math.floor(timeLeft / 60)}:{(timeLeft % 60).toString().padStart(2, '0')}</span>
                      </div>
                    </div>

                    {gameState === 'twist' && (
                      <div className="bg-orange-100 border-4 border-orange-400 rounded-xl p-6 mb-6 animate-pulse">
                        <h3 className="text-2xl font-bold text-orange-800 mb-2">🌟 PLOT TWIST! 🌟</h3>
                        <p className="text-xl text-orange-900">{currentTwist}</p>
                      </div>
                    )}

                    {currentConstraint && (
                      <div className="bg-blue-100 border-2 border-blue-400 rounded-xl p-4 mb-6">
                        <h4 className="font-bold text-blue-800">Constraint: {currentConstraint}</h4>
                      </div>
                    )}

                    <div className="bg-purple-100 rounded-xl p-6 mb-6">
                      <div className="flex items-start">
                        <Icon type="message" className="w-6 h-6 text-purple-600 mr-3 mt-1 flex-shrink-0" />
                        <p className="text-gray-800">{facilitatorMessage}</p>
                      </div>
                    </div>

                    <div className="bg-gray-50 rounded-xl p-6 mb-6 max-h-96 overflow-y-auto">
                      <h3 className="text-xl font-bold text-gray-800 mb-4">The Story So Far...</h3>
                      {story.length === 0 ? (
                        <p className="text-gray-500 italic">The story begins...</p>
                      ) : (
                        <div className="space-y-4">
                          {story.map((entry, idx) => (
                            <div key={idx} className="bg-white p-4 rounded-lg shadow">
                              <p className="font-semibold text-purple-600 mb-1">{entry.player}:</p>
                              <p className="text-gray-800">{entry.text}</p>
                            </div>
                          ))}
                        </div>
                      )}
                    </div>

                    <div className="space-y-3">
                      <textarea
                        value={storyInput}
                        onChange={(e) => setStoryInput(e.target.value)}
                        placeholder="Add your 2 sentences to the story..."
                        className="w-full px-4 py-3 border-2 border-purple-300 rounded-xl focus:outline-none focus:border-purple-500 h-24 resize-none"
                      />
                      <div className="flex gap-3">
                        <button
                          onClick={submitStory}
                          className="flex-1 bg-purple-600 text-white py-4 rounded-xl font-bold hover:bg-purple-700 transition"
                        >
                          Submit Story
                        </button>
                        {currentRound === 1 && story.length >= players.length && (
                          <button
                            onClick={introduceTwist}
                            className="flex-1 bg-orange-600 text-white py-4 rounded-xl font-bold hover:bg-orange-700 transition"
                          >
                            Introduce Twist
                          </button>
                        )}
                        {currentRound === 2 && story.length >= players.length * 2 && (
                          <button
                            onClick={startFinale}
                            className="flex-1 bg-blue-600 text-white py-4 rounded-xl font-bold hover:bg-blue-700 transition"
                          >
                            Start Finale
                          </button>
                        )}
                        {currentRound === 3 && story.length >= players.length * 3 && (
                          <button
                            onClick={endGame}
                            className="flex-1 bg-green-600 text-white py-4 rounded-xl font-bold hover:bg-green-700 transition"
                          >
                            End Game
                          </button>
                        )}
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            );
          }

          if (gameState === 'debrief') {
            return (
              <div className="min-h-screen bg-gradient-to-br from-purple-600 via-pink-500 to-orange-400 p-4">
                <div className="max-w-4xl mx-auto">
                  <div className="bg-white rounded-3xl shadow-2xl p-8">
                    <div className="text-center mb-8">
                      <h1 className="text-4xl font-bold text-gray-800 mb-2">🎉 Story Complete! 🎉</h1>
                      <p className="text-xl text-gray-600">Your team created something amazing!</p>
                    </div>

                    <div className="bg-gray-50 rounded-xl p-6 mb-8 max-h-96 overflow-y-auto">
                      <h3 className="text-2xl font-bold text-gray-800 mb-4">The Complete Story</h3>
                      <div className="space-y-4">
                        {story.map((entry, idx) => (
                          <div key={idx} className="bg-white p-4 rounded-lg shadow">
                            <p className="font-semibold text-purple-600 mb-1">{entry.player}:</p>
                            <p className="text-gray-800">{entry.text}</p>
                          </div>
                        ))}
                      </div>
                    </div>

                    <div className="bg-purple-100 rounded-xl p-6 mb-6">
                      <h3 className="text-xl font-bold text-purple-800 mb-4">Reflection Questions</h3>
                      <ul className="space-y-3 text-gray-700">
                        <li className="flex items-start">
                          <span className="text-purple-600 mr-2">•</span>
                          <span>What part of the story surprised you the most?</span>
                        </li>
                        <li className="flex items-start">
                          <span className="text-purple-600 mr-2">•</span>
                          <span>How did it feel to build on someone else's idea?</span>
                        </li>
                        <li className="flex items-start">
                          <span className="text-purple-600 mr-2">•</span>
                          <span>What made it easier or harder to keep the story coherent?</span>
                        </li>
                        <li className="flex items-start">
                          <span className="text-purple-600 mr-2">•</span>
                          <span>How does this relate to teamwork and adaptability in real work scenarios?</span>
                        </li>
                      </ul>
                    </div>

                    <button
                      onClick={() => window.location.reload()}
                      className="w-full bg-purple-600 text-white py-4 rounded-xl font-bold text-lg hover:bg-purple-700 transition"
                    >
                      Play Again
                    </button>
                  </div>
                </div>
              </div>
            );
          }
        };

        ReactDOM.render(<TeamProvGame />, document.getElementById('root'));
    </script>
</body>
</html>
