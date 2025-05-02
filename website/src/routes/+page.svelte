<script lang="ts">
    import Snake from '$lib/games/Snake.svelte';
    import Tetris from '$lib/games/Tetris.svelte';
  import { onMount } from 'svelte';
  
    let currentPath = '/home/user';
    let currentCommand = '';
    let terminalElement: HTMLDivElement;
    let isRunning = false;
    let runningProcess = '';
  
    const helpText = `Available commands:
    ls [path]     - List directory contents
    cd [path]     - Change directory
    pwd           - Print working directory
    cat [file]    - Display file contents
    clear         - Clear terminal
    play [game]   - Play a game (snake, tetris)
    help          - Display this help message
    CTRL+C        - Cancel running process`;
  
    const asciiBanner = `
  ██╗    ██╗██╗██╗     ██╗     
  ██║    ██║██║██║     ██║     
  ██║ █╗ ██║██║██║     ██║     
  ██║███╗██║██║██║     ██║     
  ╚███╔███╔╝██║███████╗███████╗
   ╚══╝╚══╝ ╚═╝╚══════╝╚══════╝
  
        WILLS WEBSITE`;
  
    let commandHistory = [
      { prompt: '', command: '', output: `<pre class="text-green-400 text-sm leading-snug font-mono">${asciiBanner}</pre>` },
      { prompt: '', command: '', output: `<pre class="whitespace-pre-wrap text-green-400 text-sm font-mono">${helpText}</pre>` }
    ];
  
    const fileSystem = {
      '/': {
        type: 'directory',
        contents: {
          home: {
            type: 'directory',
            contents: {
              user: {
                type: 'directory',
                contents: {
                  documents: {
                    type: 'directory',
                    contents: {
                      'hello.txt': {
                        type: 'file',
                        content:
                          'Hello, world! This is a text file in the documents directory.'
                      },
                      'will.txt': {
                        type: 'file',
                        content:
                          "Hey, I'm Will! I like to program computers and explore how they work."
                      },
                      'aboutThis.txt': {
                        type: 'file',
                        content:
                          'This is a simple terminal simulation built with Svelte and Tailwind CSS.'
                      },
                      'aboutme.txt': {
                        type: 'file',
                        content:
                          'I am a developer with a passion for creating robots, simulations, and interactive applications.'
                      },
                      'readme.md': {
                        type: 'file',
                        content:
                          '# Documents Directory\nThis directory contains various text files.'
                      }
                    }
                  },
                  games: {
                    type: 'directory',
                    contents: {
                      'snake': { type: 'file', content: 'A classic snake game. Use play snake to run it.' },
                      'tetris': { type: 'file', content: 'Block-stacking tetris fun! Use play tetris to run it.' }
                    }
                  },
                  '.bashrc': {
                    type: 'file',
                    content: '# .bashrc file\n# This is your bash configuration file'
                  }
                }
              }
            }
          },
          etc: {
            type: 'directory',
            contents: {
              hosts: {
                type: 'file',
                content: "127.0.0.1 localhost\n::1 localhost \n # How'd you get here?"
              }
            }
          },
          usr: {
            type: 'directory',
            contents: {
              bin: { type: 'directory', contents: {} },
              lib: { type: 'directory', contents: {} }
            }
          }
        }
      }
    };
  
    function getDirectoryFromPath(path: string) {
      if (path === '/') return fileSystem['/'];
      const parts = path.split('/').filter(Boolean);
      let current = fileSystem['/'];
      for (const part of parts) {
        const next = current.contents?.[part];
        if (!next || next.type !== 'directory') return null;
        current = next;
      }
      return current;
    }
  
    function getAbsolutePath(path: string) {
      if (path.startsWith('/')) return path;
      if (path === '.' || path === '') return currentPath;
      if (path === '..') {
        const parts = currentPath.split('/').filter(Boolean);
        parts.pop();
        return '/' + parts.join('/');
      }
      const parts = path.split('/').filter(Boolean);
      let tempPath = currentPath;
      for (const part of parts) {
        if (part === '..') {
          const tempParts = tempPath.split('/').filter(Boolean);
          tempParts.pop();
          tempPath = '/' + tempParts.join('/');
        } else if (part !== '.') {
          tempPath = tempPath === '/' ? `/${part}` : `${tempPath}/${part}`;
        }
      }
      return tempPath;
    }
  
    function getFileFromPath(path: string) {
      const absolutePath = getAbsolutePath(path);
      const parts = absolutePath.split('/').filter(Boolean);
      const filename = parts.pop();
      const dirPath = '/' + parts.join('/');
      const dir = getDirectoryFromPath(dirPath || '/');
      return dir?.contents?.[filename] || null;
    }
  
    function cancelRunning() {
      if (isRunning) {
        commandHistory = [
          ...commandHistory,
          { prompt: '', command: '^C', output: `Process '${runningProcess}' terminated.` }
        ];
        isRunning = false;
        runningProcess = '';
      }
    }
  
    function handleGameOver() {
      // This function is passed to game components to handle game over state
      commandHistory = [
        ...commandHistory,
        { prompt: '', command: '', output: 'Game over! Press CTRL+C to exit.' }
      ];
      
      setTimeout(() => {
        terminalElement.scrollTop = terminalElement.scrollHeight;
      }, 0);
    }
  
    function executeCommand(cmd: string) {
      if (isRunning && cmd.toLowerCase() !== 'ctrl+c') {
        commandHistory = [...commandHistory, { prompt: `${currentPath}$`, command: cmd, output: 'A process is already running. Press CTRL+C to cancel.' }];
        currentCommand = '';
        return;
      }
  
      const [command, ...args] = cmd.trim().split(/\s+/);
      let output = '';
  
      switch (command) {
        case 'ls': {
          const path = getAbsolutePath(args[0] || '.');
          const dir = getDirectoryFromPath(path);
          if (!dir) {
            output = `ls: cannot access '${args[0] || '.'}': No such file or directory`;
          } else {
            output = Object.entries(dir.contents)
              .sort()
              .map(([name, item]) => item.type === 'directory' ? `<span class="text-blue-400">${name}/</span>` : name)
              .join('  ');
          }
          break;
        }
  
        case 'cd': {
          const path = getAbsolutePath(args[0] || '/home/user');
          const dir = getDirectoryFromPath(path);
          if (!dir) {
            output = `cd: no such directory: ${args[0]}`;
          } else {
            currentPath = path;
          }
          break;
        }
  
        case 'pwd':
          output = currentPath;
          break;
  
        case 'cat': {
          if (!args[0]) {
            output = 'cat: missing file operand';
          } else {
            const file = getFileFromPath(args[0]);
            if (!file) {
              output = `cat: ${args[0]}: No such file or directory`;
            } else if (file.type !== 'file') {
              output = `cat: ${args[0]}: Is a directory`;
            } else {
              output = file.content;
            }
          }
          break;
        }
  
        case 'clear':
          commandHistory = [];
          currentCommand = '';
          return;
  
        case 'help':
          output = `<pre class="whitespace-pre-wrap">${helpText}</pre>`;
          break;
        
        case 'play': {
          if (!args[0]) {
            output = 'play: specify a game to play (snake, tetris)';
          } else if (!['snake', 'tetris'].includes(args[0])) {
            output = `play: '${args[0]}' not found in games`;
          } else {
            output = `Launching ${args[0]}...\n(Press CTRL+C to exit)`;
            isRunning = true;
            runningProcess = args[0];
          }
          break;
        }
  
        case 'ctrl+c':
        case '^C':
          cancelRunning();
          return;
  
        case '':
          output = '';
          break;
  
        default:
          output = `Command not found: ${command}`;
      }
  
      commandHistory = [...commandHistory, { prompt: `${currentPath}$`, command: cmd, output }];
      currentCommand = '';
  
      setTimeout(() => {
        terminalElement.scrollTop = terminalElement.scrollHeight;
      }, 0);
    }
  
    function handleKeyDown(event: KeyboardEvent) {
      // Allow game controls to work when a game is running
      // These key events will be handled by the game components
      if (isRunning && 
          (event.key === 'ArrowUp' || 
           event.key === 'ArrowDown' || 
           event.key === 'ArrowLeft' || 
           event.key === 'ArrowRight' ||
           event.key === 'w' ||
           event.key === 'a' ||
           event.key === 's' ||
           event.key === 'd' ||
           event.key === ' ')) {
        // Let the game component handle these keys
        return;
      }
  
      if (event.key === 'Enter') {
        executeCommand(currentCommand);
      } else if (event.ctrlKey && event.key.toLowerCase() === 'c') {
        executeCommand('ctrl+c');
      }
    }
  
    function focusInput() {
      const inputEl = document.getElementById('terminal-input') as HTMLInputElement;
      inputEl?.focus();
    }
    
    onMount(() => {
      focusInput();
    });

  </script>
  
  <!-- Terminal UI -->
  <div
    class="w-full h-screen bg-black text-white rounded-lg shadow-lg p-4 flex flex-col"
    on:click={focusInput}
    id="terminal-container"
  >
    <div
      bind:this={terminalElement}
      class="flex-grow overflow-auto font-mono text-green-400 text-sm p-2 space-y-2"
      style="cursor: text;"
    >
      {#each commandHistory as entry}
        <div>
          {#if entry.prompt}
            <div class="flex space-x-2">
              <span class="text-yellow-400">{entry.prompt}</span>
              <span>{entry.command}</span>
            </div>
          {/if}
          {#if entry.output}
            <div class="whitespace-pre-wrap break-words" >{@html entry.output}</div>
          {/if}
        </div>
      {/each}
  
      <!-- Game container - rendered only when a game is running -->
      {#if isRunning}
        <div class="mt-4 bg-black p-2 border border-green-400 rounded">
          {#if runningProcess === 'snake'}
            <Snake onGameOver={handleGameOver} />
          {:else if runningProcess === 'tetris'}
            <Tetris onGameOver={handleGameOver} />
          {/if}
        </div>
      {/if}
  
      <!-- Current command -->
      <div class="flex space-x-2">
        <span class="text-yellow-400">{currentPath}$</span>
        <input
          id="terminal-input"
          type="text"
          bind:value={currentCommand}
          on:keydown={handleKeyDown}
          class="bg-transparent border-none outline-none text-green-400 w-full"
        />
      </div>
    </div>
  </div>
  
  <style>
    body {
      @apply m-0 overflow-hidden;
    }
  </style>