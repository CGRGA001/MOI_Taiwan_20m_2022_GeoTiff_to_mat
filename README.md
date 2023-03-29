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
