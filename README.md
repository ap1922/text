-->service class:
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

@Service
public class DocumentService {

    @Autowired
    private DocumentRepository documentRepository;

    public List<Document> getAllFiltered(String documentType, String category, String name, String created) {
        // Convert the input created date to the format stored in the database
        SimpleDateFormat dateFormat = new SimpleDateFormat("dd-MM-yyyy");
        String formattedCreatedDate = null;
        try {
            Date date = dateFormat.parse(created);
            formattedCreatedDate = dateFormat.format(date);
        } catch (Exception e) {
            // Handle parsing exception if needed
        }

        return documentRepository.findAllByDocumentTypeAndCategoryAndNameAndCreated(documentType, category, name, formattedCreatedDate);
    }
}

