# N-Queen Visualiser

- The N-Queens puzzle is the problem of placing N chess queens on an N×N chessboard so that no two queens threaten each other. Thus, a solution requires that no two queens share the same row, column, or diagonal.

- This algorithm is designed using recursion.

![N-Queen-visualisation](visualisation.gif)

**<p align='center'>You can find the website live <a href="https://nqueen.netlify.app/">here</a></


package com.webencyclop.viper.config;

import com.webencyclop.viper.util.ViperUtils;
import org.springframework.context.ApplicationContextInitializer;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.env.ConfigurableEnvironment;
import org.springframework.core.env.MapPropertySource;
import org.springframework.core.env.PropertySource;
import org.springframework.stereotype.Component;

import javax.sql.DataSource;
import java.sql.*;
import java.util.HashMap;
import java.util.Map;

/**
 * This class loads the value from database into the Environment Properties, before application starts
 *
 * @Author Ankit Wasankar
 */
@Component
public class DBPropertyLoader implements ApplicationContextInitializer<ConfigurableApplicationContext> {
    private static final String DB_PROPS_LOADED_PROPERTY_NAME = "IS_DB_PROPS_LOADED";
    private static final String PROPERTY_SOURCE_NAME = "databaseProperties";
    private static final String QUERY = "select content_id, property_name, property_value, description, config_type, date_added from vp_config";
    private static boolean arePropertiesLoadedFromDatabase = false;

    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        if (!arePropertiesLoadedFromDatabase) {
            ConfigurableEnvironment configEnv = ((ConfigurableEnvironment) applicationContext.getEnvironment());
            PropertySource<?> appConfigProp = configEnv.getPropertySources().get("applicationConfig: [classpath:/application.properties]");
            Map<String, Object> propertySource = getPropertyMapFromDatabase(appConfigProp);
            applicationContext.getEnvironment().getPropertySources().addLast(new MapPropertySource(PROPERTY_SOURCE_NAME, propertySource));
        }
        arePropertiesLoadedFromDatabase = true;
    }

    private Map<String, Object> getPropertyMapFromDatabase(PropertySource<?> appConfigProp) {
        Map<String, Object> propertySource = new HashMap<>();
        final String url = (String) appConfigProp.getProperty("spring.datasource.url");
        final String driverClassName = (String) appConfigProp.getProperty("spring.datasource.driver");
        final String username = (String) appConfigProp.getProperty("spring.datasource.username");
        final String password = (String) appConfigProp.getProperty("spring.datasource.password");

        // Now check for database properties
        DataSource ds = null;
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet rs = null;
        try {
            if (ViperUtils.isNotBlank(driverClassName)) {
                Driver driver = (Driver) Class.forName(driverClassName).newInstance();
                DriverManager.registerDriver(driver);
            }
            connection = DriverManager.getConnection(url, username, password);
            preparedStatement = connection.prepareStatement(QUERY);
            rs = preparedStatement.executeQuery();
            // Populate all properties into the property source
            while (rs.next()) {
                final String propName = rs.getString("property_name");
                final String propValue = rs.getString("property_value");
                propertySource.put(propName, propValue);
            }
        } catch (Exception e) {
            throw new RuntimeException(e.getMessage(), e);
        } finally {
            try {
                if (rs != null) {
                    rs.close();
                }
            } catch (Exception e) {
                // supress
            }
            try {
                if (preparedStatement != null) {
                    preparedStatement.close();
                }
            } catch (Exception e) {
                // supress
            }
            try {
                if (connection != null) {
                    connection.close();
                }
            } catch (Exception e) {
                // supress
            }
        }
        return propertySource;
    }
}