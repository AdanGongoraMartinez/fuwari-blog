---
title: Mi Setup de Desarrollo en 2026
published: 2025-05-03
description: Un recorrido por mi entorno de trabajo. Fedora, Niri, Wezterm, Tmux y Neovim. Por qué elegí cada herramienta y cómo las hago trabajar juntas.
image: "./setup.png"
tags: [Linux, Neovim, Tmux, Workflow, Personal]
category: Tech
---

# Mi Setup de Desarrollo en 2026

Después de cuatro años de experimentos, pruebas y muchos errores, este es el entorno que uso todos los días para programar.

## El Editor: Neovim

Todo empezó por curiosidad, viendo los videos de ThePrimeagen. Al principio fue confuso e incómodo, pero se sentía como estar jugando un videojuego mientras editaba código.

Cada vez que integraba una nueva función de las vim motions a mi workflow, se sentía como aprender un combo en un juego de pelea. Ahora estoy en el punto donde me resulta más cómodo usar Neovim que cualquier otro editor.

### Configuración

Uso LazyVim con algunos ajustes personales.

Tema:

```lua
  { "ellisonleao/gruvbox.nvim" },

  -- Configure LazyVim to load gruvbox
  {
    "LazyVim/LazyVim",
    opts = {
      colorscheme = "gruvbox",
    },
  }
```

Statusline personalizada:

```lua
-- Muestra: modo, branch, diff, filename, filetype, diagnostics

  "nvim-lualine/lualine.nvim",
  -- Carga perezosa después de que casi todo esté cargado
  event = "VeryLazy",

  -- La dependencia de devicons es necesaria para filetype/iconos
  dependencies = { "nvim-tree/nvim-web-devicons" },

  init = function()
    -- Guardar el estado original de 'laststatus'
    vim.g.lualine_laststatus = vim.o.laststatus
    if vim.fn.argc(-1) > 0 then
      -- Si se abren archivos (argc > 0), usar una línea de estado vacía hasta que lualine cargue.
      vim.o.statusline = " "
    else
      -- Si no se abren archivos (dashboard), ocultar la línea de estado inmediatamente.
      vim.o.laststatus = 0
    end
  end,

  opts = function()
    local lualine_require = require("lualine_require")
    lualine_require.require = require

    local icons = LazyVim.config.icons

    -- Restaurar 'laststatus' al valor guardado
    vim.o.laststatus = vim.g.lualine_laststatus

    local opts = {
      options = {
        -- Usar 'auto' para seguir el colorscheme 'gruvbox'
        theme = "auto",
        globalstatus = false,
        icons_enabled = true,

        -- **SOLUCIÓN LAZYVIM PARA OCULTAR EN DASHBOARD/BIENVENIDA:**
        -- Esto deshabilita Lualine en los filetypes del dashboard/bienvenida
        disabled_filetypes = { statusline = { "dashboard", "alpha", "ministarter", "snacks_dashboard" } },

        section_separators = {
          left = "\u{e0bc}",
          right = "\u{e0ba}",
        },
        component_separators = {},
      },
      sections = {
        -- **TU CONFIGURACIÓN PERSONALIZADA**
        lualine_a = { { "mode", right_padding = 2 } },
        lualine_b = { "branch" },
        lualine_c = {
          "%=", -- Centrar
          "diff",
          {
            "filename",
            file_status = true,
            path = 1,
            shorting_target = 40,
          },
          {
            "filetype",
            icon_only = true,
            separator = "",
            padding = {
              left = 1,
              right = 0,
            },
          },
          {
            "diagnostics",
            symbols = {
              error = icons.diagnostics.Error,
              warn = icons.diagnostics.Warn,
              info = icons.diagnostics.Info,
              hint = icons.diagnostics.Hint,
            },
          },
          {
            function()
              local reg = vim.fn.reg_recording()
              if reg == "" then
                return ""
              end
              return "󰑋 Grabando @" .. reg -- Puedes cambiar el icono
            end,
            color = { fg = "#ff9e64", gui = "bold" }, -- Un color naranja/llamativo
          },
        },
        lualine_x = {
          -- Aquí van los componentes complejos de LazyVim/Snacks (Noice, DAP, Lazy updates, Diff)
          -- Mantendré la estructura de LazyVim para lualine_x, ya que es compleja.
          Snacks.profiler.status(),
          -- ... (componentes de Noice, DAP, Lazy updates, Diff de tu código original) ...

          -- Si no usas Snacks, Noice ni DAP, lualine_x puede ser más simple:
          -- {
          --   "diff",
          --   symbols = { added = icons.git.added, modified = icons.git.modified, removed = icons.git.removed },
          -- },
        },
        lualine_y = {
          function()
            local current_line = vim.fn.line(".")
            local total_lines = vim.fn.line("$")
            return string.format("%d | %d", current_line, total_lines)
          end,
        },
        lualine_z = {
          { "location", left_padding = 2 },
        },
      },
      inactive_sections = {
        lualine_a = {},
        lualine_b = {
          "%=", -- Centrar
          {
            "diagnostics",
            symbols = {
              error = icons.diagnostics.Error,
              warn = icons.diagnostics.Warn,
              info = icons.diagnostics.Info,
              hint = icons.diagnostics.Hint,
            },
          },
          {
            "filename",
            file_status = true,
            path = 1,
            shorting_target = 40,
          },
          {
            "filetype",
            icon_only = true,
            separator = "",
            padding = {
              left = 1,
              right = 0,
            },
          },
          "location",
        },
        lualine_c = {},
        lualine_x = {},
        lualine_y = {},
        lualine_z = {},
      },
      extensions = { "neo-tree", "lazy", "fzf" },
    }

    -- La lógica de integración de 'trouble.nvim' debe ir aquí si deseas mantenerla.

    return opts
  end,
```

### Mis Keybindings Principales

```lua
-- Mapeos para moverte entre ventanas (splits) en Modo Normal (n)
vim.keymap.set("n", "<C-Left>", "<C-w>h", { desc = "Moverse a la ventana izquierda" })
vim.keymap.set("n", "<C-Down>", "<C-w>j", { desc = "Moverse a la ventana inferior" })
vim.keymap.set("n", "<C-Up>", "<C-w>k", { desc = "Moverse a la ventana superior" })
vim.keymap.set("n", "<C-Right>", "<C-w>l", { desc = "Moverse a la ventana derecha" })

vim.keymap.set("n", "<C-Left>", "<Cmd>TmuxNavigateLeft<CR>", { desc = "Moverse a la ventana izquierda" })
vim.keymap.set("n", "<C-Down>", "<Cmd>TmuxNavigateDown<CR>", { desc = "Moverse a la ventana inferior" })
vim.keymap.set("n", "<C-Up>", "<Cmd>TmuxNavigateUp<CR>", { desc = "Moverse a la ventana superior" })
vim.keymap.set("n", "<C-Right>", "<Cmd>TmuxNavigateRight<CR>", { desc = "Moverse a la ventana derecha" })

-- Mapeos para redimensionar ventanas usando Alt + flechas (A es Alt)
vim.keymap.set("n", "<A-Left>", "<cmd>vertical resize -2<cr>", { desc = "Disminuir ancho" })
vim.keymap.set("n", "<A-Right>", "<cmd>vertical resize +2<cr>", { desc = "Aumentar ancho" })
vim.keymap.set("n", "<A-Up>", "<cmd>resize +2<cr>", { desc = "Aumentar alto" })
vim.keymap.set("n", "<A-Down>", "<cmd>resize -2<cr>", { desc = "Disminuir alto" })

-- Mapeos para centar la pantalla al hacer scroll
vim.keymap.set("n", "<C-d>", "<C-d>zz", { desc = "" })
vim.keymap.set("n", "<C-u>", "<C-u>zz", { desc = "" })
vim.keymap.set("n", "n", "nzzzv", { desc = "" })
vim.keymap.set("n", "N", "Nzzzv", { desc = "" })

-- Mapeos para moverse entre tabs usando p y n
vim.keymap.set("n", "<leader><tab>p", "<cmd>tabPrevious<cr>", { desc = "Previous Tab" })
vim.keymap.set("n", "<leader><tab>n", "<cmd>tabNext<cr>", { desc = "Next Tab" })
```

## El Multiplexor: Tmux

Un paso natural después de acostumbrarme a Neovim. Tener un multiplexor cambia por completo el workflow. Trabajar en múltiples proyectos y tener muchas herramientas de terminal al mismo tiempo se volvió muy fácil y rápido.

Lo único malo son los keybindings por defecto, muy poco intuitivos.

### Configuración Clave

```tmux
# Cambiar prefix a `
unbind C-b
set-option -g prefix `

####################################################
# Navigation & Keys
####################################################

# Split panes
bind '\' split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"

# Reload config
bind r source-file ~/.tmux.conf \; display-message "󰑓 Config reloaded"

# Navigation (Vim style)
set -g @vim_navigator_mapping_left "C-Left"
set -g @vim_navigator_mapping_right "C-Right"
set -g @vim_navigator_mapping_up "C-Up"
set -g @vim_navigator_mapping_down "C-Down"

# Window navigation (Alt + n/p)
bind -n M-n next-window
bind -n M-p previous-window

# Copy Mode (vi)
set-window-option -g mode-keys vi
bind-key v copy-mode
bind-key -T copy-mode-vi v send-keys -X begin-selection
bind-key -T copy-mode-vi y send-keys -X copy-selection-and-cancel
```

````
####################################################
# Status Bar Design (Evil Style)
####################################################

set-option -g status-position top
set -g status-style "bg=#{@gb_bg},fg=#{@gb_fg}"
set -g status-justify left

# --- Left Side (Spacing) ---
set -g status-left " #[fg=#{@gb_blue}]  "
set -g status-left-length 20

# --- Windows List ---
# Inactiva
set -g window-status-format "#[fg=#{@gb_fg_dim}]#I:#W "
# Ventana Activa con indicador de Zoom
set -g window-status-current-format "#[bold]  #W #{?window_zoomed_flag,󰊓 ,}"
set -g window-status-current-style "#{?window_zoomed_flag,fg=#{@gb_orange},fg=#{@gb_blue}}"

# --- Right Side (Git & Session) ---
# Mostrando Rama de Git y Nombre de Sesión
set -g status-right-length 60
set -g status-right "#[fg=#{@gb_fg_dim}] #(cd #{pane_current_path} 2>/dev/null && git rev-parse --abbrev-ref HEAD 2>/dev/null || echo 'none') #[fg=#{@gb_fg_dim}]| #[fg=#{@gb_blue},bold]󰇄 #S "

####################################################
# Panes & Selection
####################################################
set -g pane-border-style "fg=#{@gb_bg_alt}"
set -g pane-active-border-style "fg=#{@gb_orange}"

# Mensajes de comando
set -g message-style "bg=#{@gb_orange},fg=#{@gb_bg},bold"

Statusline:
```tmux
####################################################
# Status Bar Design (Evil Style)
####################################################

set-option -g status-position top
set -g status-style "bg=#{@gb_bg},fg=#{@gb_fg}"
set -g status-justify left

# --- Left Side (Spacing) ---
set -g status-left " #[fg=#{@gb_blue}]  "
set -g status-left-length 20

# --- Windows List ---
# Inactiva
set -g window-status-format "#[fg=#{@gb_fg_dim}]#I:#W "
# Ventana Activa con indicador de Zoom
set -g window-status-current-format "#[bold]  #W #{?window_zoomed_flag,󰊓 ,}"
set -g window-status-current-style "#{?window_zoomed_flag,fg=#{@gb_orange},fg=#{@gb_blue}}"

# --- Right Side (Git & Session) ---
# Mostrando Rama de Git y Nombre de Sesión
set -g status-right-length 60
set -g status-right "#[fg=#{@gb_fg_dim}] #(cd #{pane_current_path} 2>/dev/null && git rev-parse --abbrev-ref HEAD 2>/dev/null || echo 'none') #[fg=#{@gb_fg_dim}]| #[fg=#{@gb_blue},bold]󰇄 #S "

####################################################
# Panes & Selection
####################################################
set -g pane-border-style "fg=#{@gb_bg_alt}"
set -g pane-active-border-style "fg=#{@gb_orange}"

# Mensajes de comando
set -g message-style "bg=#{@gb_orange},fg=#{@gb_bg},bold"

````

## El Emulador: Wezterm

Mi terminal Preferida. ¿Por qué? Porque usa Lua para la configuración, al igual que Neovim. Sí, eso es todo.

He probado otros emuladores como Kitty y Ghostty. Siento que los tres son igual de buenos y rápidos, pero poder tener configuraciones en el mismo lenguaje que Neovim me resulta más cómodo.

## El Sistema Operativo: Fedora

Mi introducción a Linux comenzó con Linux Mint, pero pasé por varias distribuciones y experimenté mucho. En la laptop que usé durante la universidad tuve Arch Linux con GNOME la mayor parte del tiempo.

Sin embargo, siempre tuve el miedo de que algo se rompiera justo el día que necesitara usar la laptop. Como no quería ese riesgo en mi PC del trabajo, decidí usar Fedora.

Siento que es una excelente distribución. Mantiene paquetes modernos con una base estable que se actualiza cada seis meses. Ahora mismo considero Fedora con KDE Plasma la distro por excelencia para nuevos usuarios. Fácil de usar pero que otorga mucho control.

## El Gestor de Ventanas: Niri

En mis tiempos con Arch Linux probé múltiples gestores de ventanas y siempre tuve curiosidad por los tiling window managers. Sin embargo, no me acomodaba con ninguno y volvía a GNOME cada vez que encontraba un problema.

Para cuando empecé a usar Fedora en mi PC actual, decidí instalarle Cosmic DE como entorno secundario. Fue una experiencia más agradable de lo que esperaba. No perfecta pero si lo suficientemente buena para un proyecto tan joven.

Por otro lado, me empezaron a salir videos sobre un nuevo window manager en YouTube: un scrolling window manager. Leí la documentación y, tomando la recomendación de usar Dank Material Shell (DMS), lo instalé.

Ha sido todo lo que siempre quise. Rápido, bonito, cómodo y ligero. No es perfecto, hay algunos problemas visuales, pero son muy pocos y muy infrecuentes.

## Herramientas que Abandoné

- **VS Code**: Tiene muy buenos plugins y soporte, pero es pesado y lento. Además, la UI tiene demasiadas cosas. Prefiero quedarme con mi entorno minimalista en la terminal.
- **Zed**: VS Code pero rápido y hecho en Rust. Muy bueno, pero Neovim + Tmux es demasiado cómodo para cambiarlo.
- **Zellij**: Una alternativa a Tmux escrita en Rust. Es muy rápida, pero la UI no es de mi agrado y los keybindings se llegan a sobreponer con los de Neovim.

## Mi Workflow Típico

Usualmente uso Tmuxifier para generar sesiones usando scripts. Solo necesito ejecutar:

```bash
tmuxifier load-session <session>
```

Y automáticamente tengo el layout que necesito para el proyecto.

### Un Día Típico

1. Abrir Wezterm
2. Lanzar la sesión de tmux:

   ```bash
   tmuxifier load-session <session>
   ```

3. El comando automáticamente ejecuta el proyecto y abre las herramientas que necesito
4. Realizar cambios usando Neovim
5. Usar LazyGit para control de versiones

Mis sesiones de Tmux suelen tener 3 ventanas compartidas: Neovim, una para ejecutar el proyecto y otra con OpenCode. En muchos casos agrego un split horizontal a la ventana que ejecuta el proyecto para ejecutar pruebas unitarias. Dependiendo del proyecto, agrego más ventanas.

## Filosofía

Mi setup sigue una filosofía centrada en:

- **Velocidad**: Todo está optimizado para minimizar fricción entre pensar y ejecutar
- **Control**: Cada aspecto del entorno está configurado a mi medida
- **Diversión**: Disfruto el proceso de construir y mejorar mi workflow
- **Minimalismo**: Solo mantengo lo que realmente uso

No le recomiendo este setup a alguien que no le guste probar distintas herramientas y configuraciones o que no esté acostumbrado a usar la terminal. Este entorno requiere inversión de tiempo para configurarlo y mantenerlo, y no es para todos.

## Conclusión

Este setup es el resultado de años de experimentos y curiosidad. No es perfecto, pero funciona exactamente como lo necesito. Y al final, eso es lo que importa: encontrar las herramientas que te hagan más productivo y disfruta el proceso.

:::note
Si quieres ver mi configuración completa, está disponible en mis [dotfiles](https://github.com/AdanGongoraMartinez/dotfiles.git).
:::
