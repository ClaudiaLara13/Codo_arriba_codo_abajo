# Codo_arriba_codo_abajo
Codigo codo arriba codo abajo, coordenada inicial cero 


function gui_cinematica_inversa()
    % --- Ventana Principal ---
    fig = figure('Name', 'Cinemática Inversa - Codo Arriba / Codo Abajo', ...
                 'NumberTitle', 'off', 'Position', [100, 100, 850, 500]);

    % --- Panel de Control ---
    pnl = uipanel('Title', ' Parámetros de Entrada ', 'Position', [0.02, 0.05, 0.32, 0.9]);

    % Selección de Configuración (Inicia en Indefinido)
    uicontrol(pnl, 'Style', 'text', 'String', 'Configuración del Codo:', ...
              'Position', [10, 420, 240, 20], 'HorizontalAlignment', 'left');
    pop_codo = uicontrol(pnl, 'Style', 'popupmenu', ...
                         'String', {'-- Indefinido --', 'Codo Arriba (Izquierda)', 'Codo Abajo (Derecha)'}, ...
                         'Value', 1, ...
                         'Position', [10, 390, 240, 30], 'Callback', @calcular_y_graficar);

    % Entradas para Coordenada X_F (Inicia en 0.0)
    uicontrol(pnl, 'Style', 'text', 'String', 'Coordenada X_F:', ...
              'Position', [10, 340, 240, 20], 'HorizontalAlignment', 'left');
    txt_xf = uicontrol(pnl, 'Style', 'edit', 'String', '0.0', ...
                       'Position', [10, 315, 240, 25], 'Callback', @calcular_y_graficar);

    % Entradas para Coordenada Y_F (Inicia en 0.0)
    uicontrol(pnl, 'Style', 'text', 'String', 'Coordenada Y_F:', ...
              'Position', [10, 270, 240, 20], 'HorizontalAlignment', 'left');
    txt_yf = uicontrol(pnl, 'Style', 'edit', 'String', '0.0', ...
                       'Position', [10, 245, 240, 25], 'Callback', @calcular_y_graficar);

    % Longitudes de los eslabones l1 y l2 (Inician en 0.0)
    uicontrol(pnl, 'Style', 'text', 'String', 'Longitud l1:', ...
              'Position', [10, 200, 110, 20], 'HorizontalAlignment', 'left');
    txt_l1 = uicontrol(pnl, 'Style', 'edit', 'String', '0.0', ...
                       'Position', [10, 175, 110, 25], 'Callback', @calcular_y_graficar);

    uicontrol(pnl, 'Style', 'text', 'String', 'Longitud l2:', ...
              'Position', [140, 200, 110, 20], 'HorizontalAlignment', 'left');
    txt_l2 = uicontrol(pnl, 'Style', 'edit', 'String', '0.0', ...
                       'Position', [140, 175, 110, 25], 'Callback', @calcular_y_graficar);

    % Caja para mostrar Resultados (Inicia en Indefinido)
    txt_resultado = uicontrol(pnl, 'Style', 'text', 'String', '', ...
                              'Position', [10, 10, 240, 140], ...
                              'HorizontalAlignment', 'left', 'FontSize', 10, 'BackgroundColor', [0.94 0.94 0.94]);

    % --- Ejes Gráficos ---
    ax = axes('Parent', fig, 'Position', [0.4, 0.12, 0.55, 0.8]);
    grid(ax, 'on'); axis(ax, 'equal');
    xlabel(ax, 'X'); ylabel(ax, 'Y');
    title(ax, 'Representación Geométrica del Robot 2R');

    % Mostrar el estado inicial indefinido al abrir
    calcular_y_graficar();

    % --- Función Callback ---
    function calcular_y_graficar(~, ~)
        % 1. Leer valores de la GUI
        X_F = str2double(get(txt_xf, 'String'));
        Y_F = str2double(get(txt_yf, 'String'));
        l1  = str2double(get(txt_l1, 'String'));
        l2  = str2double(get(txt_l2, 'String'));
        opcion_codo = get(pop_codo, 'Value'); % 1: Indefinido, 2: Codo Arriba, 3: Codo Abajo

        % Validar si la configuración del codo está indefinida
        if opcion_codo == 1
            res_str = sprintf([' Configuración: Indefinida\n\n' ...
                               ' sigma = Indefinido\n' ...
                               ' beta  = Indefinido\n' ...
                               ' gamma = Indefinido\n\n' ...
                               ' theta1 = Indefinido\n' ...
                               ' theta2 = Indefinido']);
            set(txt_resultado, 'String', res_str);
            cla(ax); 
            hold(ax, 'on');
            % Ejes en negro en el origen (Plano inicial)
            plot(ax, [-5, 5], [0, 0], '-k', 'LineWidth', 1.2, 'HandleVisibility', 'off');
            plot(ax, [0, 0], [-5, 5], '-k', 'LineWidth', 1.2, 'HandleVisibility', 'off');
            grid(ax, 'on'); xlim(ax, [-5, 5]); ylim(ax, [-5, 5]);
            hold(ax, 'off');
            return;
        end

        % Validar entradas no numéricas
        if isnan(X_F) || isnan(Y_F) || isnan(l1) || isnan(l2)
            errordlg('Por favor, ingresa únicamente valores numéricos válidos.', 'Error de Entrada');
            return;
        end

        % 2. Geometría (Pizarrón)
        r_cuadrado = X_F^2 + Y_F^2;
        r = sqrt(r_cuadrado);

        % --- VERIFICACIÓN DE LÍMITES FÍSICOS Y ADVERTENCIAS ---
        if l1 <= 0 || l2 <= 0
            errordlg('Las longitudes de los eslabones (l1 y l2) deben ser mayores a cero.', 'Eslabones Inválidos');
            set(txt_resultado, 'String', ' Error: Longitudes de eslabón igual a cero.');
            cla(ax); grid(ax, 'on');
            return;
        elseif r > (l1 + l2)
            msg = sprintf('¡Punto fuera de alcance!\n\nLa distancia al objetivo es r = %.2f, pero el alcance máximo es l1 + l2 = %.2f.', r, l1 + l2);
            errordlg(msg, 'Fuera de Alcance');
            set(txt_resultado, 'String', ' Error: Punto demasiado lejano.');
            cla(ax); grid(ax, 'on');
            return;
        elseif r < abs(l1 - l2)
            msg = sprintf('¡Punto demasiado cercano!\n\nLa distancia r = %.2f es menor a la diferencia de los eslabones |l1 - l2| = %.2f.', r, abs(l1 - l2));
            errordlg(msg, 'Auto-colisión / Zona Muerta');
            set(txt_resultado, 'String', ' Error: Punto demasiado cercano.');
            cla(ax); grid(ax, 'on');
            return;
        elseif r == 0
            errordlg('El objetivo está en el origen (0,0). Ingrese un punto fuera del origen.', 'Indeterminación');
            set(txt_resultado, 'String', ' Error: Punto en origen.');
            cla(ax); grid(ax, 'on');
            return;
        end

        sigma = atan2d(Y_F, X_F); % sigma = tan^-1(Y_F / X_F)

        % Ley de Cosenos (Pizarrón)
        cos_gamma = (l1^2 + l2^2 - r_cuadrado) / (2 * l1 * l2);
        cos_gamma = max(-1, min(1, cos_gamma));
        gamma = acosd(cos_gamma);

        cos_beta = (l1^2 + r_cuadrado - l2^2) / (2 * l1 * r);
        cos_beta = max(-1, min(1, cos_beta));
        beta = acosd(cos_beta);

        % 3. Métodos exactos según la opción
        if opcion_codo == 2
            % --- CODO ARRIBA (Izquierda) ---
            theta1 = beta + sigma;          % theta1 = beta + sigma
            theta2 = 180 - gamma;           % theta2 = 180° - gamma
            theta2_abs = theta1 - 180 + gamma;
            nombre_config = 'Codo Arriba';
        elseif opcion_codo == 3
            % --- CODO ABAJO (Derecha) ---
            theta1 = sigma - beta;          % theta1 = sigma - beta
            theta2 = 180 - gamma;           % theta2 = 180° - gamma
            theta2_abs = theta1 + 180 - gamma;
            nombre_config = 'Codo Abajo';
        end

        % 4. Coordenadas de las articulaciones
        x0 = 0; y0 = 0;
        x1 = l1 * cosd(theta1);
        y1 = l1 * sind(theta1);
        
        x2 = x1 + l2 * cosd(theta2_abs);
        y2 = y1 + l2 * sind(theta2_abs);

        % 5. Desplegar resultados
        res_str = sprintf([' Configuración: %s\n\n' ...
                           ' sigma = %.2f°\n' ...
                           ' beta  = %.2f°\n' ...
                           ' gamma = %.2f°\n\n' ...
                           ' theta1 = %.2f°\n' ...
                           ' theta2 = %.2f°'], ...
                          nombre_config, sigma, beta, gamma, theta1, theta2);
        set(txt_resultado, 'String', res_str);

        % 6. Redibujar en los Ejes
        cla(ax);
        hold(ax, 'on');
        
        lim = l1 + l2 + 1;

        % --- LÍNEAS DE LOS EJES CARTESIANOS EN NEGRO (Plano X e Y) ---
        plot(ax, [-lim, lim], [0, 0], '-k', 'LineWidth', 1.2, 'HandleVisibility', 'off'); % Eje X
        plot(ax, [0, 0], [-lim, lim], '-k', 'LineWidth', 1.2, 'HandleVisibility', 'off'); % Eje Y
        
        % Línea r (referencia)
        plot(ax, [0, X_F], [0, Y_F], '--k', 'LineWidth', 1);
        
        % Eslabones
        plot(ax, [x0, x1], [y0, y1], '-o', 'LineWidth', 3, 'MarkerSize', 8, ...
             'Color', [0.85 0.325 0.098], 'MarkerFaceColor', 'r'); % Eslabón 1
        plot(ax, [x1, x2], [y1, y2], '-o', 'LineWidth', 3, 'MarkerSize', 8, ...
             'Color', [0 0.447 0.741], 'MarkerFaceColor', 'b');    % Eslabón 2
        
        xlim(ax, [-lim/2, lim]); ylim(ax, [-lim/2, lim]);
        grid(ax, 'on'); axis(ax, 'equal');
        legend(ax, {'Línea r', 'Eslabón 1 (l1)', 'Eslabón 2 (l2)'}, ...
               'Location', 'northeast');
        hold(ax, 'off');
    end
end
