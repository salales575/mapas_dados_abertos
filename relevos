import geopandas # Importa a biblioteca GeoPandas para trabalhar com dados geoespaciais
import matplotlib.pyplot as plt # Importa Matplotlib para criar gráficos estáticos
import folium # Importa Folium para criar mapas interativos
import ssl # Importa o módulo SSL para lidar com certificados de segurança (usado aqui para uma solução paliativa)

# --- Configurações Iniciais e Carregamento de Dados ---
# URL do serviço WFS (Web Feature Service) do IBGE que fornece dados GeoJSON de geomorfologia 1:5.000 (CREN).
# WFS permite acessar feições geográficas vetoriais.
wfs_url = "https://www.geoservicos.ibge.gov.br/geoserver/wms?service=WFS&version=1.0.0&request=GetFeature&typeName=CREN:geomorfologia_5000&outputFormat=JSON"

try:
    print(f"Tentando carregar dados do WFS: {wfs_url}")

    # --- Solução Paliativa para o Erro SSL (CUIDADO: Reduz a segurança!) ---
    # Este trecho sobrescreve o contexto SSL padrão para permitir conexões
    # HTTPS sem verificar o certificado. Isso pode resolver erros de certificado
    # em alguns ambientes, mas NÃO é recomendado para produção devido a riscos de segurança.
    ssl._create_default_https_context = ssl._create_unverified_context

    # Carrega os dados GeoJSON diretamente da URL usando geopandas.read_file().
    # Ele lê o conteúdo da URL, que é um arquivo GeoJSON, e o converte em um GeoDataFrame.
    gdf = geopandas.read_file(wfs_url)

    # --- Verificação e Exibição de Informações Básicas dos Dados ---
    print(f"Dados carregados com sucesso! Projeção (CRS): {gdf.crs}") # Imprime o Sistema de Coordenadas de Referência (CRS)
    print(f"Número de feições: {len(gdf)}") # Imprime a quantidade de registros (polígonos) carregados
    print(f"Primeiras 5 linhas dos dados:\n{gdf.head()}") # Exibe as primeiras 5 linhas do GeoDataFrame para inspeção

    # --- Plotagem de Mapa Estático com GeoPandas e Matplotlib ---
    print("\nGerando mapa estático com Matplotlib...")
    # Cria uma figura e um eixo para o plot. figsize define o tamanho da imagem.
    fig, ax = plt.subplots(1, 1, figsize=(12, 12))

    # Verifica se a coluna 'NOMEL_1' existe no GeoDataFrame.
    # Esta coluna provavelmente contém o nome do tipo geomorfológico, útil para categorizar e colorir.
    if 'NOMEL_1' in gdf.columns:
        # Plota o GeoDataFrame, colorindo as feições com base na coluna 'NOMEL_1'.
        # `cmap='Spectral'` define o mapa de cores.
        # `edgecolor='black'` define a cor da borda dos polígonos.
        # `legend=True` exibe a legenda para as categorias.
        gdf.plot(column='NOMEL_1', ax=ax, cmap='Spectral', edgecolor='black', legend=True)
        ax.set_title('Geomorfologia (CREN) - IBGE (por NOMEL_1)') # Define o título do mapa
    else:
        # Se 'NOMEL_1' não existir, plota sem categorização específica, usando um mapa de cores padrão.
        gdf.plot(ax=ax, cmap='viridis', edgecolor='black', legend=True)
        ax.set_title('Geomorfologia (CREN) - IBGE') # Título genérico

    ax.set_xlabel('Longitude') # Define o rótulo do eixo X
    ax.set_ylabel('Latitude') # Define o rótulo do eixo Y
    plt.show() # Exibe o mapa estático

    # --- Plotagem de Mapa Interativo com Folium ---
    print("\nGerando mapa interativo com Folium...")
    # Verifica se o GeoDataFrame não está vazio antes de tentar criar o mapa interativo.
    if not gdf.empty:
        gdf_wgs84 = gdf
        # Verifica se o CRS (Sistema de Coordenadas de Referência) do GeoDataFrame já existe e se é WGS84 (EPSG:4326).
        # Se não for WGS84, ele reprojeta os dados para este CRS, que é o padrão para Folium e mapas web.
        if gdf.crs:
            if gdf.crs.to_epsg() != 4326:
                gdf_wgs84 = gdf.to_crs(epsg=4326)
        else:
            # Se o CRS não estiver definido, ele o define como WGS84.
            gdf_wgs84 = gdf.set_crs(epsg=4326, allow_override=True)

        # Encontra geometrias válidas (não nulas) para calcular o centro do mapa.
        valid_geometries = gdf_wgs84.geometry.dropna()
        if not valid_geometries.empty:
            # Calcula o centróide médio das geometrias válidas para centralizar o mapa.
            center_lat = valid_geometries.centroid.y.mean()
            center_lon = valid_geometries.centroid.x.mean()
        else:
            # Se não houver geometrias válidas, usa as coordenadas de Brasília como centro padrão.
            print("Nenhuma geometria válida encontrada para calcular o centróide. Usando centro padrão (Brasília).")
            center_lat, center_lon = -15.7801, -47.9292 # Coordenadas de Brasília

        # Cria um objeto de mapa Folium, centralizado nas coordenadas calculadas e com zoom inicial.
        # `tiles='OpenStreetMap'` define a camada base do mapa.
        m = folium.Map(location=[center_lat, center_lon], zoom_start=6, tiles='OpenStreetMap')

        # Prepara os campos (colunas) que serão exibidos no tooltip (informações ao passar o mouse).
        # Exclui a coluna 'geometry' que não é informativa para o tooltip.
        tooltip_fields = [col for col in gdf_wgs84.columns if col != 'geometry']

        # Adiciona as feições geomorfológicas ao mapa Folium como uma camada GeoJSON.
        # `gdf_wgs84.to_json()` converte o GeoDataFrame para o formato GeoJSON.
        # `folium.features.GeoJsonTooltip` configura o tooltip para exibir os dados das colunas selecionadas.
        folium.GeoJson(
            gdf_wgs84.to_json(),
            name='Geomorfologia', # Nome da camada no controle de camadas
            tooltip=folium.features.GeoJsonTooltip(fields=tooltip_fields, aliases=tooltip_fields, localize=True)
        ).add_to(m) # Adiciona a camada ao mapa

        # Adiciona um controle de camadas ao mapa, permitindo ligar/desligar a camada "Geomorfologia".
        folium.LayerControl().add_to(m)

        # Salva o mapa interativo em um arquivo HTML.
        mapa_interativo_nome = 'mapa_geomorfologia_ibge_interativo.html'
        m.save(mapa_interativo_nome)
        print(f"Mapa interativo de geomorfologia IBGE gerado com sucesso! Verifique o arquivo '{mapa_interativo_nome}'.")

    else:
        print("GeoDataFrame vazio. Não foi possível gerar o mapa interativo.")

# --- Tratamento de Erros ---
except Exception as e:
    # Captura e imprime qualquer erro que ocorra durante a execução do código.
    print(f"Ocorreu um erro ao carregar os dados ou gerar o mapa: {e}")
    print("\nPossíveis causas adicionais e soluções:")
    print("1. Verifique sua conexão com a internet.")
    print("2. Tente acessar a URL diretamente no seu navegador: https://www.geoservicos.ibge.gov.br/geoserver/wms?service=WFS&version=1.0.0&request=GetFeature&typeName=CREN:geomorfologia_5000&outputFormat=JSON")
    print("   Se o navegador também mostrar um aviso de segurança, o problema é no servidor ou na sua rede.")
    print("3. Se você estiver em uma rede corporativa, pode haver um proxy ou firewall interferindo. Consulte seu administrador de rede.")
    print("4. Certifique-se de que todas as bibliotecas (`geopandas`, `matplotlib`, `folium`) estão instaladas e atualizadas.")
