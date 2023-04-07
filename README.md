# MOI_Taiwan_20m_2022_GeoTiff_to_mat
+ 自「 https://www.tgos.tw/TGOS/Web/MetaData/TGOS_Query_MetaData.aspx?key=TW-06-301000000A-612709 」下載的「DEMg_tawiwan_20m_20220406_g14.tif」與「DEMg_tawiwan_20m_20220406_g14.tfw」取出XYZ資料。
+ 水平解析度:20，單位[公尺]，座標系統:WGS84；
+ 高程解析度:浮點數，單位[公尺]，高程原點為海平面，向上為正。
+ 過程中順便展示地形圖，以利確認資料範圍。

### 程式碼
+ MOI_Taiwan_20m_2022_GeoTiff_to_mat_v20230328a.m
```matlab
%**************************************************************************
%   Name: MOI_Taiwan_20m_2022_GeoTiff_to_mat_v20230323a.m
%   Copyright:  
%   Author: HsiupoYeh 
%   Version: v20230328a
%   Description:自「https://www.tgos.tw/TGOS/Web/MetaData/TGOS_Query_MetaData.aspx?key=TW-06-301000000A-612709」下載的「DEMg_tawiwan_20m_20220406_g14.tif」與「DEMg_tawiwan_20m_20220406_g14.tfw」取出XYZ資料。
%       水平解析度:20，單位[公尺]，座標系統:WGS84；
%       高程解析度:浮點數，單位[公尺]，高程原點為海平面，向上為正。
%       過程中順便展示地形圖，以利確認資料範圍。
%   需求檔案:       
%       DEMg_tawiwan_20m_20220406_g14.tif(下載自https://www.tgos.tw/TGOS/Web/MetaData/TGOS_Query_MetaData.aspx?key=TW-06-301000000A-612709)，
%       DEMg_tawiwan_20m_20220406_g14.tfw(下載自https://www.tgos.tw/TGOS/Web/MetaData/TGOS_Query_MetaData.aspx?key=TW-06-301000000A-612709)，
%       該檔案必須放置於工作目錄下的資料夾「MOI_Taiwan_20m_2022_GeoTiff」中。
%**************************************************************************
    clear;clc;close all
    %--
    % 讀GeoTiff檔
    tic
    dem_geotiff_data=imread('MOI_Taiwan_20m_2022_GeoTiff\DEMg_tawiwan_20m_20220406_g14.tif');
    toc
    % <18852x10035 single>
    % Elapsed time is 1.924751 seconds.
    %--
    tic
    %--
    dem_geotiff_tfw=load('MOI_Taiwan_20m_2022_GeoTiff\DEMg_tawiwan_20m_20220406_g14.tfw');
    toc
    % Elapsed time is 0.000482 seconds.
    %--
    % 取出長寬
    [dem_geotiff_info.Height,dem_geotiff_info.Width]=size(dem_geotiff_data);
    %
    disp(dem_geotiff_info.Width)
    %        10035
    disp(dem_geotiff_info.Height)
    %        18852
    %--
    % World file(*.tfw) 
    % Line 1: A: x-component of the pixel width (x-scale)
    % Line 2: D: y-component of the pixel width (y-skew)
    % Line 3: B: x-component of the pixel height (x-skew)
    % Line 4: E: y-component of the pixel height (y-scale), typically negative
    % Line 5: C: x-coordinate of the center of the original image's upper left pixel transformed to the map
    % Line 6: F: y-coordinate of the center of the original image's upper left pixel transformed to the map
    disp(dem_geotiff_tfw)
    %           20
    %            0
    %            0
    %          -20
    %       150980
    %      2799160    
    %--
    dem_X_vector=(0:dem_geotiff_info.Width-1)*dem_geotiff_tfw(1)+dem_geotiff_tfw(5);
    disp('dem_X_vector:')
    disp(num2str(dem_X_vector(1:5)))
    % 150980  151000  151020  151040  151060
    dem_Y_vector=(0:(dem_geotiff_info.Height-1))*dem_geotiff_tfw(4)+dem_geotiff_tfw(6);
    disp('dem_Y_vector:')
    disp(num2str(dem_Y_vector(1:5)))
    % 2799160  2799140  2799120  2799100  2799080
    %--
    % 嘗試繪製成圖形，interp資料恰好是XYZ資料
    [xi,yi] = meshgrid(dem_X_vector,dem_Y_vector);
    zi=zeros(size(xi));
    ci_interp=double(dem_geotiff_data);
    surf(xi,yi,zi,ci_interp,'FaceColor','interp','EdgeColor','none')
    % 調整colorbar
    [temp_demcmap_cmap,temp_demcmap_clim]=demcmap(ci_interp,1000);
    colormap(temp_demcmap_cmap)
    % 顯示colorbar。
    color_bar_handle=colorbar('location','eastoutside');
    %--
    % 設定colorbar標題文字
    color_bar_title_handle=title(color_bar_handle,'Elevation[m]');
    % 調整其他繪圖參數(順序好像有影響)
    axis equal
    box on
    view(0,90)
    xlabel('TWD97 E[m]')
    ylabel('TWD97 N[m]')    
    % 標題
    title({'內政部DTM地形圖(不含水深)';'水平解析度: 20公尺';'垂直解析度: 單精度浮點數(單位:公尺)'})
    %--
    % 另存成mat檔案
    % 符合DEM的XYZ檔案格式
    % 普通XYZ格式如下:
    % xyz =
    %      1     1     1
    %      1     2     2
    %      1     3     3
    %      1     4     4
    %      2     1     5
    %      2     2     6
    %      2     3     7
    %      2     4     8
    %      3     1     9
    %      3     2    10
    %      3     3    11
    %      3     4    12
    %      4     1    13
    %      4     2    14
    %      4     3    15
    %      4     4    16
    %      5     1    17
    %      5     2    18
    %      5     3    19
    %      5     4    20
    %--
    % 可用dem_xyz=sortrows(xyz,[-2,1,3])排序為標準DEM格式，但效率極差。
    % 標準DEM的XYZ格式如下:
    %--
    % dem_xyz =
    % 
    %      1     4     4
    %      2     4     8
    %      3     4    12
    %      4     4    16
    %      5     4    20
    %      1     3     3
    %      2     3     7
    %      3     3    11
    %      4     3    15
    %      5     3    19
    %      1     2     2
    %      2     2     6
    %      3     2    10
    %      4     2    14
    %      5     2    18
    %      1     1     1
    %      2     1     5
    %      3     1     9
    %      4     1    13
    %      5     1    17
    %--
    % 快速轉置GeoTiff資料恰好可以排出標準DEM的XYZ格式
    xi=xi';
    yi=yi';
    ci_interp=ci_interp';
    MOI_Taiwan_20m_DEM_2022.Data.XYZ=[xi(:) yi(:) ci_interp(:)];
    MOI_Taiwan_20m_DEM_2022.Data.XYZ_Header={'TWD97_E[m]','TWD97_N[m]','Elevation[m]'};
    %--
    % 補充資訊
    MOI_Taiwan_20m_DEM_2022.Description='來源:自「https://www.tgos.tw/TGOS/Web/MetaData/TGOS_Query_MetaData.aspx?key=TW-06-301000000A-612709」下載的「DEMg_tawiwan_20m_20220406_g14.tif」與「DEMg_tawiwan_20m_20220406_g14.tfw」取出XYZ資料。水平解析度:20，單位[公尺]，座標系統:WGS84；高程解析度:浮點數，單位[公尺]，高程原點為海平面，向上為正。過程中順便展示地形圖，以利確認資料範圍。';
    MOI_Taiwan_20m_DEM_2022.Version='20230328a';
    MOI_Taiwan_20m_DEM_2022.Editor='HsiupoYeh';
    % 存檔
    if ~(exist('Output','dir')==7)
        mkdir('Output')
    end
    save('Output\MOI_Taiwan_20m_DEM_2022.mat','MOI_Taiwan_20m_DEM_2022','-v7.3')
    saveas(gca,'Output\MOI_Taiwan_20m_DEM_2022.png')
    %--
    disp('完成!')
```
### 完整釋出版本
+ https://github.com/CGRGA001/MOI_Taiwan_20m_2022_GeoTiff_to_mat/releases/tag/v20230328a
  + 使用MATLAB2014a測試通過。

### 程式碼
+ MOI_Taiwan_20m_2022_GeoTiff_to_mat_v20230331a.m
```matlab
%**************************************************************************
%   Name: MOI_Taiwan_20m_2022_mat_plot_v20230331a.m
%   Copyright:  
%   Author: HsiupoYeh 
%   Version: v20230331a
%   Description:利用整理好的「MOI_Taiwan_20m_DEM_2022.mat」繪製台灣地形圖(不含水深)。
%       水平解析度:20，單位[公尺]，座標系統:TWD97；
%       高程解析度:單精度浮點數，單位[公尺]，高程原點為海平面，向上為正。
%   需求檔案:       
%       MOI_Taiwan_20m_DEM_2022.mat(從DEMg_tawiwan_20m_20220406_g14.tif萃取出來的)，
%       該檔案必須放置於工作目錄下的資料夾「MOI_Taiwan_20m_DEM_2022」中。
%**************************************************************************
    clear;clc;close all
    % 指定擷取中心點經緯度
WGS84_Longitude_in_degrees=121.547627;
WGS84_Latitude_in_degrees=25.175558;
    [TWD97_E_in_meters,TWD97_N_in_meters]=Snyder_Formula_WGS84toTWD97_121(WGS84_Longitude_in_degrees,WGS84_Latitude_in_degrees);
    % 指定範圍
E_left_distance_in_meters=1000*10;
E_right_distance_in_meters=1000*10;
E_top_distance_in_meters=1000*10;
E_bottom_distance_in_meters=1000*10;
    TWD97_E_min=TWD97_E_in_meters-E_left_distance_in_meters;
    TWD97_E_max=TWD97_E_in_meters+E_right_distance_in_meters;
    TWD97_N_max=TWD97_N_in_meters+E_top_distance_in_meters;
    TWD97_N_min=TWD97_N_in_meters-E_bottom_distance_in_meters;
    %--
    % 讀GeoTiff檔
    tic
    temp_data=load('MOI_Taiwan_20m_DEM_2022\MOI_Taiwan_20m_DEM_2022.mat');
    toc
    % Elapsed time is 10.651541 seconds.
    %--
    tic
    %--
    %
    disp(temp_data.MOI_Taiwan_20m_DEM_2022.Description)
    % 來源:自「https://www.tgos.tw/TGOS/Web/MetaData/TGOS_Query_MetaData.aspx?key=TW-06-301000000A-612709」下載的「DEMg_tawiwan_20m_20220406_g14.tif」與「DEMg_tawiwan_20m_20220406_g14.tfw」取出XYZ資料。
    % 水平解析度:20，單位[公尺]，座標系統:WGS84；高程解析度:浮點數，單位[公尺]，高程原點為海平面，向上為正。
    % 過程中順便展示地形圖，以利確認資料範圍。水平解析度:1-arc-minutes (~1.8 kilometers)，單位[度]，座標系統:WGS84；高程解析度:整數，單位[公尺]，高程原點為海平面，向上為正。
    target_index=(temp_data.MOI_Taiwan_20m_DEM_2022.Data.XYZ(:,1)>=TWD97_E_min) & ...
                 (temp_data.MOI_Taiwan_20m_DEM_2022.Data.XYZ(:,1)<=TWD97_E_max) & ...
                 (temp_data.MOI_Taiwan_20m_DEM_2022.Data.XYZ(:,2)>=TWD97_N_min) & ...
                 (temp_data.MOI_Taiwan_20m_DEM_2022.Data.XYZ(:,2)<=TWD97_N_max);
    temp_data.MOI_Taiwan_20m_DEM_2022.Data.XYZ=temp_data.MOI_Taiwan_20m_DEM_2022.Data.XYZ(target_index,:);
    %--
    % 重新排序(如果不確定是不是依照標準DEM的XYZ格式排序，則可以使用此方法，但效率極差，通常對應標準資料不需要此步驟)
    % temp_data.ETOPO1_Taiwan_DEM.Data.XYZ=sortrows(temp_data.MOI_Taiwan_20m_DEM_2022.Data.XYZ,[-2,1,3]);
    %--
    % 計算X與Y方向的像素點數量(因為是Grid註冊，像素中心點都坐落在Tick上)
    X_Tick_count=sum(temp_data.MOI_Taiwan_20m_DEM_2022.Data.XYZ(:,2)==temp_data.MOI_Taiwan_20m_DEM_2022.Data.XYZ(1,2));
    disp(['X_Tick_count = ',num2str(X_Tick_count)])
    Y_Tick_count=sum(temp_data.MOI_Taiwan_20m_DEM_2022.Data.XYZ(:,1)==temp_data.MOI_Taiwan_20m_DEM_2022.Data.XYZ(1,1));
    disp(['Y_Tick_count = ',num2str(Y_Tick_count)])
    %--
    % 整理資料
    dem_xi=reshape(temp_data.MOI_Taiwan_20m_DEM_2022.Data.XYZ(:,1),X_Tick_count,[])';
    dem_yi=reshape(temp_data.MOI_Taiwan_20m_DEM_2022.Data.XYZ(:,2),X_Tick_count,[])';
    dem_zi=zeros(size(dem_xi));
    dem_ci_interp=reshape(temp_data.MOI_Taiwan_20m_DEM_2022.Data.XYZ(:,3),X_Tick_count,[])';
    dem_ci_interp(dem_ci_interp==-32767)=NaN;
    %--
    % 繪圖
    surf(dem_xi,dem_yi,dem_zi,dem_ci_interp,'FaceColor','interp','EdgeColor','none')
    % 這裡限制僅有測試過的版本可運行。
    MATLAB_Version=version;
    if strcmp(MATLAB_Version,'7.8.0.347 (R2009a)')
        disp('提示:正確運行在MATLAB R2009a版本...')
        MATLAB_Version_str='R2009a';
    elseif strcmp(MATLAB_Version,'8.3.0.532 (R2014a)')
        disp('提示:正確運行在MATLAB R2014a版本...')
        MATLAB_Version_str='R2014a'; 
    elseif strcmp(MATLAB_Version,'8.4.0.150421 (R2014b)')
        disp('提示:正確運行在MATLAB R2014b版本...')
        MATLAB_Version_str='R2014b'; 
    else
        disp('錯誤:未測試通過的MATLAB版本,return!')
        ExportEdiPNG.Error.String='錯誤:未測試通過的MATLAB版本,return!';
        return
    end
    %--
    % 調整colorbar
    if (strcmp(MATLAB_Version_str,'R2009a') || strcmp(MATLAB_Version_str,'R2014a'))
        Elevation_Ticks=[0:10:1200];%注意，舊版的MATLAB(2014a之前)colormap內資料超過1000可能會在輸出圖片的時候沒反應。這裡故意減少數量。
        Elevation_Ticks_count=length(Elevation_Ticks);
        [my_land_colormap,my_land_demcmap_clim]=demcmap(Elevation_Ticks,Elevation_Ticks_count);
        Elevation_Ticks=[-1000:10:-1];%注意，舊版的MATLAB(2014a之前)colormap內資料超過1000可能會在輸出圖片的時候沒反應。這裡故意減少數量。
        Elevation_Ticks_count=length(Elevation_Ticks);
        [my_sea_colormap,my_sea_demcmap_clim]=demcmap(Elevation_Ticks,Elevation_Ticks_count);
        temp_demcmap_cmap=[my_sea_colormap;my_land_colormap];
        colormap(temp_demcmap_cmap)%注意，舊版的MATLAB(2014a之前)colormap內資料超過1000可能會在輸出圖片的時候沒反應。
    elseif strcmp(MATLAB_Version,'8.4.0.150421 (R2014b)')
        Elevation_Ticks=[0:1200];
        Elevation_Ticks_count=length(Elevation_Ticks);
        [my_land_colormap,my_land_demcmap_clim]=demcmap(Elevation_Ticks,Elevation_Ticks_count);
        Elevation_Ticks=[-1000:-1];%注意，舊版的MATLAB(2014a之前)colormap內資料超過1000可能會在輸出圖片的時候沒反應。這裡故意減少數量。
        Elevation_Ticks_count=length(Elevation_Ticks);
        [my_sea_colormap,my_sea_demcmap_clim]=demcmap(Elevation_Ticks,Elevation_Ticks_count);
        temp_demcmap_cmap=[my_sea_colormap;my_land_colormap];
        colormap(temp_demcmap_cmap)%注意，舊版的MATLAB(2014a之前)colormap內資料超過1000可能會在輸出圖片的時候沒反應。
    end    
    set(gca,'Clim',[-1000,1200])
    % 顯示colorbar。
    color_bar_handle=colorbar('location','eastoutside');
    %--
    % 設定colorbar標題文字
    color_bar_title_handle=title(color_bar_handle,'Elevation[m]');
    % 調整其他繪圖參數(順序好像有影響)
    axis equal
    box on
    view(0,90)
    xlabel('TWD97 E[m]')
    ylabel('TWD97 N[m]')   
    % 標題
    title({'內政部DTM地形圖(不含水深)';'水平解析度: 20公尺';'垂直解析度: 單精度浮點數(單位:公尺)'})
    %
    %--
    % 存檔
    if ~(exist('Output','dir')==7)
        mkdir('Output')
    end
    saveas(gca,'Output\MOI_Taiwan_20m_DEM_2022.png')
    %--
```
### 完整釋出版本
+ https://github.com/CGRGA001/MOI_Taiwan_20m_2022_GeoTiff_to_mat/releases/tag/v20230331a
  + 使用MATLAB2014a測試通過。
